+++
title = 'Golang Error Wrapping'
date = 2026-02-22T11:38:40+01:00
tags = ["golang", "errors"]
technologies = ["golang"]
+++

Error handling in go is great!
Verbose, but great.
While it took some getting used to, by now I much prefer to deal with errors as values, rather than throwing/catching them.
While it can feel verbose to keep writing the `if err != nil` check multiple times in the same routine it is still unbeaten in readability and clarity, I find.

But there is a use case, that I have been struggling with a little bit.
That use case of retaining full error information in the application while providing clear descriptive error messages to the user, while not leaking internal application error messages, revealing technical implementation details.
This is a use case I have encounter many times in "layered" architectures (MVC, hexagonal, ...).

## Problem example

I have a microservice with an application core containing the domain logic.
We connect a grpc adapter as the client interface and a database adapter to fetch and persist data.
All of the components should be as loosely coupled as possible with generic interfaces.
That way we can easily maintain or replace the components individually.

Now if an there is an error in the repository adapter, there will be a very technical error message about what is going on.
Let's say for example, a foreign key constraint was violated.
As a user I try to create a child entity for a parent that does not exist, I expect an error from the application with a clear, average person readable message of what the problem is.

E.g. I try create a book (`/v1/{parent=publishers/*}/books`) for a publisher (`/v1/publishers/{id}`) that is not in our database.
The application should probably tell me something like `error: publisher 'xyz' does not exist`.
Along with the error, I return a grpc `codes.InvalidArgument` (or an http Bad Request for REST) so that the client app can also understand the error-kind.

I can figure out what is going on in the db adapter and construct a clear error message and pass the new error up the chain.
This makes sense, because we have handled the error where it occurred and don't have to figure out what to make of the original error in some other layer that should not care about the technical details of what happened.
I would then be the job of the job of the left adapter to pass that message to the client with the correct error code.
But knowing what that error is, is tricky, as the message is constructed to provide parameterized information and can't easily be checked for error what kind of error this is.

What I used to do, was to just use generalized error messages such as `error: resource does not exists` in global variables in the repo adapter package and return that error.
One can then easily use `errors.Is()` to check what the source error kind is and set the status code accordingly in another package.
But we loose the option to provide the user with more details.

Enter: wrapping errors.
I still use general global error like this. 

```golang
var (
	ErrInvalidReference = errors.New("invalid reference")
	ErrAlreadyExists    = errors.New("already exists")
	ErrNotFound         = errors.New("not found")
    // ...
	ErrUnknownDbError   = errors.New("unknown db error")
)
```

This is a very common pattern in golang.
I make sure, that I put those in a package decoupled from the adapter implementation, so I can reuse them across multiple potential implementations of that adapter.

In the adapter itself, I use error wrapping to wrap this general error in a more detailed error message

```golang {linenos=false}
return fmt.Errorf("%w: book references publisher '%s' that does not exist", 
    ErrInvalidReference, book.Publisher.Id)
```

Now, even though we have our parameterized error message with useful details for the user, we can easily check what the src wrapped error was, and thus set the correct error code.

```golang
func codeFromError(err error) codes.Code {
	if errors.Is(err, app.ErrUnauthorized) {
		return codes.PermissionDenied
	}

	if errors.Is(err, entity.ErrNotFound) {
		return codes.NotFound
	}

	if errors.Is(err, entity.ErrAlreadyExists) {
		return codes.AlreadyExists
	}

	if errors.Is(err, entity.ErrInvalidReference) {
		return codes.InvalidArgument
	}

	return codes.Unknown
}
```

So in our left adapter, we call the application core and if the call returns an error we check for the wrapped error and set the code accordingly.

So with these wrapped error "constants" we have an interface between for adapters.
If any error they return wraps one of the given constants we can easily handle it without having to analyze the "actual" parameterized error message.
