package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "os"
    "sync"
    "time"
)

// Service represents a service to be monitored
type Service struct {
    Name string `json:"name"`
    URL  string `json:"url"`
}

func main() {
    // Read JSON file
    data, err := ioutil.ReadFile("services.json")
    if err != nil {
        fmt.Println("Error reading file:", err)
        os.Exit(1)
    }

    var services []Service
    if err := json.Unmarshal(data, &services); err != nil {
        fmt.Println("Error parsing JSON:", err)
        os.Exit(1)
    }

    var wg sync.WaitGroup

    for _, service := range services {
        wg.Add(1)
        go func(s Service) {
            defer wg.Done()
            checkService(s)
        }(service)
    }

    wg.Wait()
}

// checkService checks if the service is up or down
func checkService(s Service) {
    client := http.Client{
        Timeout: 5 * time.Second,
    }

    resp, err := client.Get(s.URL)
    if err != nil {
        fmt.Printf("[DOWN]  %s (%s): %s\n", s.Name, s.URL, err.Error())
        return
    }
    defer resp.Body.Close()

    if resp.StatusCode == http.StatusOK {
        fmt.Printf("[UP]    %s (%s)\n", s.Name, s.URL)
    } else {
        fmt.Printf("[DOWN]  %s (%s) - Status Code: %d\n", s.Name, s.URL, resp.StatusCode)
    }
}
