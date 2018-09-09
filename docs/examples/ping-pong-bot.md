## Ping-Pong bot
So the time has come where you want to be a bot engineer huh? In this article you are introduced to creating the most basic bot. This snippet will contain the main function's body.


```GoLang
// create a channel to listen for termination signals (graceful shutdown)
termSignal := make(chan os.Signal, 1)
signal.Notify(termSignal, syscall.SIGINT, syscall.SIGTERM, os.Interrupt, os.Kill)

// create a Disgord session
session, err := disgord.NewSession(&disgord.Config{
    Token: os.Getenv("DISGORD_TOKEN"),
})
if err != nil {
    panic(err)
}

// create a handler and bind it to new message events
session.AddListener(event.KeyMessageCreate, func(session Session, data *event.MessageCreate) {
    msg := data.Message

    fmt.Printf("user{%s} said: `%s`\n", msg.Author.Username, msg.Content) // noob logging
    if msg.Content == "ping" {
        msg.RespondString(session, "pong")
        // alternative: session.SendMsgString(msg.ChannelID, "pong")
    }
})

// connect to the discord gateway to receive events
err = session.Connect()
if err != nil {
    panic(err)
}

// Keep the socket connection alive, until you terminate the application
<-termSignal
session.Disconnect()
```
If sending the response is slow. You are most likely rate limited, and Disgord is trying to wait out the rate limit delay before the message is sent. If you want to get an error message in stead of waiting enable the config option upon session creation, `Config.CancelRequestWhenRateLimited`. Note that if the bot has to wait a time that is longer than the http.Client.Timeout value, it always returns an error saying you were rate limited.