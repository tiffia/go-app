# Use the official Golang image as a builder
FROM golang:1.23.4 AS builder
# Set the working directory inside the container
WORKDIR /app
# Copy the Go module files and download dependencies
COPY go.mod go.sum ./
RUN go mod download
# Copy the rest of the application code
COPY . .
# Build the Go application
RUN go build -o go-app
# Use a smaller base image for the final container
FROM debian:bookworm-slim
# Set the working directory in the new container
WORKDIR /app
# Copy the compiled binary from the builder
COPY --from=builder /app/go-app .
# Expose the port the app runs on
EXPOSE 8080
# Run the application
CMD ["./go-app"]