# zerologlib

This project wraps [`zerolog`](https://github.com/rs/zerolog) to provide a way to configure `Logger`'s after creation.

## Setup

Each package should create their own logger instance using `NewLogger`:

```golang
var Logger = log.Root.NewLogger(
    "github.com/pastdev/myapp/pkg/myapp",
    func(name string, lgr zerolog.Logger) zerolog.Logger {
        return lgr.With().Str("logger", name).Logger()
    })
```

Then in your application startup code you would:

```golang
// could be loaded from a configuration file...
logFormat := "json" // or "pretty"
logLevel := "debug"
logWriter := os.Stderr

// ...

lvl, err := zerolog.ParseLevel(logLevel)
if err != nil {
    return fmt.Errorf("parse level: %w", err)
}

var writer io.Writer
switch logFormat {
case "pretty":
    writer = zerolog.ConsoleWriter{Out: logWriter}
default:
    writer = logWriter
}

// set the root logger to use the configured logger at the configured level
// and add a timestamp to each message
log.Root.Configure(
    writer,
    func(name string, lgr zerolog.Logger) zerolog.Logger {
        return lgr.With().Timestamp().Logger().Level(lvl)
    })

// tune the util package to WarnLevel
log.Configure(
    "github.com/pastdev/myapp/pkg/util",
    func(name string, lgr zerolog.Logger) zerolog.Logger {
        return lgr.Level(zerolog.WarnLevel)
    })
```

The default root logger is a no-op logger so that makes this safe to include in libraries without unintentionally bleeding log output into stdout/stderr.
