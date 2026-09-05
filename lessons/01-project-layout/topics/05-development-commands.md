# Development commands

## Why they matter

A small, repeatable command set makes local checks understandable and gives reviewers the same way to verify both services. Run these commands from the assigned service module directory.

## Commands

### `go mod tidy`

Adds module requirements needed by the source code and removes unused requirements. It may modify `go.mod` and `go.sum`, so review and commit those changes when they are intentional.

### `go fmt ./...`

Formats Go source files in every package in the module. It modifies files whose formatting differs from the Go standard format.

### `go vet ./...`

Reports selected suspicious constructs in every package. It is a useful static check, but it is not a complete linter and does not prove that the program is correct.

### `go build ./...`

Compiles every package and command in the module. For this form of the command, the goal is to verify compilation rather than produce a named deployment binary.

## Applying it to Lesson 1

Document these commands in the assigned service README and make sure they succeed before requesting review. A `Makefile`, task runner, additional linter, and automated test command are not required in this lesson.

## Further reading

- [Go command documentation](https://pkg.go.dev/cmd/go) describes the Go tool's commands and package patterns.
- [Go Modules Reference: `go mod tidy`](https://go.dev/ref/mod#go-mod-tidy) defines how module requirements are updated.
