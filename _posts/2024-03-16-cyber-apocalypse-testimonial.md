---
layout: post
title:  "CA CTF - Testmimonial"
date:   2024-03-16 20:22:00 -0600
categories: ctf cyberapocalypse
tags: htb cyberapocalypse ctf web grpc
description: "Writeup/Solution to Cyber Apocalypse 2024 CTF: Testimonial"
---

## Scenario

Files provided from HTB are in the [ctf assets](/assets/ctfs).

This challenge was interesting. Two different ports were provided to access the ctf on, in the provided docker files they are `1337` (which appears to usually be http for HTB) and `50045`, which is unknown.

Opening up `localhost:1337` after starting the docker container shows us this page.

![testimonial homepage](/assets/images/ctf/ca2024/testimonial-homepage.png)

Messing around a bit, shows you can submit testimonials to the website using the form at the bottom of the page. The testimonials will then show up next to the default 3 shown.


### Normal behaviour

First we want to figure out how the application works.

The main go file has this:

```go
func main() {
	router := chi.NewMux()

	router.Handle("/*", http.StripPrefix("/", http.FileServer(http.FS(FS))))
	router.Get("/", handler.MakeHandler(handler.HandleHomeIndex))
	go startGRPC()
	log.Fatal(http.ListenAndServe(":1337", router))
}

type server struct {
	pb.RickyServiceServer
}

func startGRPC() error {
	lis, err := net.Listen("tcp", ":50045")
	if err != nil {
		log.Fatal(err)
	}
	s := grpc.NewServer()

	pb.RegisterRickyServiceServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatal(err)
	}
	return nil
}
```

This indicates that we do indeed have a [go http server](https://pkg.go.dev/net/http) on port `1337` and the other port exposed is a `50045` which is a [GRPC port](https://en.wikipedia.org/wiki/GRPC).

The homepage is handled by `handler/home.go` via the following code:

```go
package handler

import (
	"htbchal/client"
	"htbchal/view/home"
	"net/http"
)

func HandleHomeIndex(w http.ResponseWriter, r *http.Request) error {
	customer := r.URL.Query().Get("customer")
	testimonial := r.URL.Query().Get("testimonial")
	if customer != "" && testimonial != "" {
		c, err := client.GetClient()
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)

		}

		if err := c.SendTestimonial(customer, testimonial); err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)

		}
	}
	return home.Index().Render(r.Context(), w)
}
```

The core of this code is checking for `customer` and `testimonial` parameters; If they exist, it means the user was submitting a new testimonial, otherwise it just renders the homepage (which is a golang server side render).

Digging into the testimonial submission, we see that we are making a new client and submitting the customer/testimonial data. We look at the client initialized from `c, err := client.GetClient()` and see this relavent code:

```go
type Client struct {
	pb.RickyServiceClient
}

func GetClient() (*Client, error) {
	mutex.Lock()
	defer mutex.Unlock()

	if grpcClient == nil {
		conn, err := grpc.Dial(fmt.Sprintf("127.0.0.1%s", ":50045"), grpc.WithInsecure())
		if err != nil {
			return nil, err
		}

		grpcClient = &Client{pb.NewRickyServiceClient(conn)}
	}

	return grpcClient, nil
}

func (c *Client) SendTestimonial(customer, testimonial string) error {
	ctx := context.Background()
	// Filter bad characters.
	for _, char := range []string{"/", "\\", ":", "*", "?", "\"", "<", ">", "|", "."} {
		customer = strings.ReplaceAll(customer, char, "")
	}

	_, err := c.SubmitTestimonial(ctx, &pb.TestimonialSubmission{Customer: customer, Testimonial: testimonial})
	return err
}
```

This is a gRPC client that connects to the grpc port we are hosting and calls the `SubmitTestimonial` RPC function defined in the `ptypes.proto` file:

```proto
service RickyService {
    rpc SubmitTestimonial(TestimonialSubmission) returns (GenericReply) {}
}

message TestimonialSubmission {
    string customer = 1;
    string testimonial = 2;
}

message GenericReply {
    string message = 1;
}
```

The proto file shows what functions can be defined but not what the 

### Finding the Injection Point
https://templ.guide/
Looking at the code, there 