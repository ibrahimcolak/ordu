```go
package main

import (
    "context"
    "fmt"
    "io"
    "io/ioutil"

    texttospeech "cloud.google.com/go/texttospeech/apiv1"
    "cloud.google.com/go/texttospeech/apiv1/texttospeechpb"
)

// synthesizeTextFile reads a text file and uses the Text-to-Speech API to synthesize the
// text as a file.
func synthesizeTextFile(w io.Writer, filename string) error {
    // filename := "testdata/test.txt"
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return fmt.Errorf("ReadFile: %v", err)
    }
    text := string(data)

    ctx := context.Background()

    // Creates a client.
    client, err := texttospeech.NewClient(ctx)
    if err != nil {
        return fmt.Errorf("NewClient: %v", err)
    }
    defer client.Close()

    req := &texttospeechpb.SynthesizeSpeechRequest{
        // Select the text to synthesize.
        Input: &texttospeechpb.SynthesisInput{
            InputSource: &texttospeechpb.SynthesisInput_Text{Text: text},
        },
        // Build the voice request.
        Voice: &texttospeechpb.VoiceSelectionParams{
            LanguageCode: "en-US",
            SsmlGender:   texttospeechpb.SsmlVoiceGender_FEMALE,
        },
        // Select the type of audio file you want returned.
        AudioConfig: &texttospeechpb.AudioConfig{
            AudioEncoding: texttospeechpb.AudioEncoding_MP3,
        },
    }

    // Performs the text-to-speech request.
    resp, err := client.SynthesizeSpeech(ctx, req)
    if err != nil {
        return fmt.Errorf("SynthesizeSpeech: %v", err)
    }

    // The resp's AudioContent is binary.
    err = ioutil.WriteFile("output.mp3", resp.AudioContent, 0644)
    if err != nil {
        return fmt.Errorf("WriteFile: %v", err)
    }
    fmt.Fprintf(w, "Audio content written to file: output.mp3\n")
    return nil
}
  
```
