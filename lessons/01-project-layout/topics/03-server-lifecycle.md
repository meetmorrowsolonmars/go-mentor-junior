# HTTP server lifecycle

## Why it matters

A service must start predictably and stop without abandoning active requests. When it receives a termination signal, graceful shutdown closes the server's listeners first, so new connections are no longer accepted. It then waits for active requests to finish, up to a fixed deadline, before the process exits.

## Startup and configuration

Startup code should:

1. read the listen address from `HTTP_ADDR`, using a documented default such as `:8080`;
2. construct providers and application services;
3. register handlers;
4. create one explicit `http.Server`;
5. start serving and wait for either a signal or a server error.

Read configuration once during startup and pass values to the code that needs them. Domain code and handlers should not repeatedly read environment variables. For Lesson 1, the shutdown deadline can be a named constant; a configuration framework and server-timeout tuning are unnecessary.

## Graceful shutdown

This focused example omits route construction, configuration parsing, and logging:

```go
const shutdownTimeout = 10 * time.Second

signalCtx, stop := signal.NotifyContext(
	context.Background(),
	os.Interrupt,
	syscall.SIGTERM,
)
defer stop()

serveErr := make(chan error, 1)
go func() {
	serveErr <- server.ListenAndServe()
}()

select {
case err := <-serveErr:
	if errors.Is(err, http.ErrServerClosed) {
		return nil
	}
	return err
case <-signalCtx.Done():
	shutdownCtx, cancel := context.WithTimeout(
		context.Background(),
		shutdownTimeout,
	)
	defer cancel()

	if err := server.Shutdown(shutdownCtx); err != nil {
		return err
	}

	err := <-serveErr
	if errors.Is(err, http.ErrServerClosed) {
		return nil
	}
	return err
}
```

`ListenAndServe` normally returns `http.ErrServerClosed` after `Shutdown`; that value is not an unexpected service failure. `Shutdown` uses its context as the maximum wait for active connections, so the process must wait for it to return.

## Applying it to Lesson 1

The service must have one executable and one HTTP server. It must respond to an interactive interrupt and `SIGTERM`, stop accepting connections, wait for graceful shutdown, and report unexpected startup or shutdown errors. Automated shutdown tests are deferred to the testing lesson.

## Further reading

- [`http.Server`](https://pkg.go.dev/net/http#Server) and [`Server.Shutdown`](https://pkg.go.dev/net/http#Server.Shutdown) define the server lifecycle behavior.
- [`signal.NotifyContext`](https://pkg.go.dev/os/signal#NotifyContext) connects operating-system signals to context cancellation.
- [The Twelve-Factor App: Disposability](https://12factor.net/disposability) gives the operational motivation for graceful termination.
