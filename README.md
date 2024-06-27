# Annoy-go

A go binding of Spotify's Annoy (Approximate Nearest Neighbors Oh Yeah). 
Codes are generated using Swig from the lastest(by now) Annoy commit(2be37c9e015544be2cf60c431f0cccc076151a2d).

## Build

CGO is required to build this project. Use the following command to add it to your go.mod:

```
go get github.com/oligo/annoy-go@latest
```

## API Usage

Swig does not generates document for the go source. Please consult the documents of Spotify Annoy for API usage.
Here is a simple example located in the example directory to demostrate the basic usage. 

```
package main

import (
	"errors"
	"fmt"
	"os"

	annoyindex "github.com/oligo/annoy-go"
)

const indexFile = "./embeddings.ann"

func main() {
	embeddings := [][]float32{
		{0.1, 0.2, 0.31},
		{0.4, 0.55, 0.67},
	}

	index := annoyindex.NewAnnoyIndexAngular(len(embeddings[0]))
	if PathExists(indexFile) {
		if ret := index.Load(indexFile, false); !ret {
			panic("Failed to load file without prefault")
		}
	} else {
		defer func() {
			index.Build(5)
			index.Save(indexFile, false)
		}()
		for i, embedding := range embeddings {
			index.AddItem(i, embedding)
		}
	}

	distance1 := index.GetDistance(0, 1)
	distance2 := index.GetDistance(0, 2)
	fmt.Printf("distance between 1 and 2: %f, 0 and 3: %f\n", distance1, distance2)

	// query with a vector
	query := []float32{0.12, 0.22, 0.3}
	result := []int{}
	distances := []float32{}
	index.GetNnsByVector(query, 10, -1, &result, &distances)

	for i, idx := range result {
		vector := []float32{}
		index.GetItem(idx, &vector)
		fmt.Printf("found vector: %v, distance: %f\n", result, distances[i])
	}
}

func PathExists(path string) bool {
	stat, err := os.Stat(path)
	if errors.Is(err, os.ErrNotExist) {
		return false
	}

	if err == nil || stat != nil {
		return true
	}

	return false
}
```