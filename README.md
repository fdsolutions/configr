# configr

[![Build Status](https://travis-ci.org/adrianduke/configr.svg?branch=master)](https://travis-ci.org/adrianduke/configr) [![Coverage Status](https://coveralls.io/repos/adrianduke/configr/badge.svg?branch=master&service=github)](https://coveralls.io/github/adrianduke/configr?branch=master) [![GoDoc](https://godoc.org/github.com/adrianduke/configr?status.svg)](https://godoc.org/github.com/adrianduke/configr)

Configr provides an abstraction above configuration sources, allowing you to use a single interface to expect and get all your configuration values.

**Features:**
- **Single interface for configuration values:** Simple API (Get(), String(), Bool()...)
- **Extendable config sources:** Load config from a file, database, environmental variables or any source you can get data from
- **Multiple source support:** Add as many sources as you can manage, FILO merge strategy employed (first source added has highest priority)
- **Nested Key Support:** `production.payment_gateway.public_key` `production.payment_gateway.private_key`
- **Value validation support:** Any matching key from every source is validated by your custom validators
- **Required keys support:** Ensure keys exist after parsing, otherwise error out
- **Blank config generator:** Register as many keys as you need and use the blank config generator
- **Custom blank config encoder support:** Implement an encoder for any data format and have a blank config generated in it
- **Type conversion support:** Your config has string "5" but you want an int 5? No problem
- **Comes pre-baked with JSON and TOML file support**

Built for a project at [HomeMade Digital](http://homemadedigital.com/), configrs primary goal was to eliminate user error when deploying projects with heavy configuration needs. The inclusion of required key support, value validators, descriptions and blank config generator allowed us to reduce pain for seperated client ops teams when deploying our apps. Our secondary goal was flexible configuration sources be it pulling from Mongo Document, DynamoDB Table, JSON or TOML files.

## Example

### Simple JSON File

Pre register some keys to expect from your configuration sources (typically via init() for small projects, or via your own object initiation system for larger projects):
```go
	configr.RequireKey("email.fromAddress", "Email from address")
	configr.RequireKey("email.subject", "Email subject")
	configr.RegisterKey("email.retryOnFail", "Retry sending email if it fails", false)
	configr.RegisterKey("email.maxRetries", "How many times to retry email resending", 3)
```

Create some configuration:
```json
{
	"email": {
		"fromAddress": "my@email.com",
		"subject": "A Subject",
		"retryOnFail": true,
		"maxRetries": 5
	}
}
```

Add a source:
```go
	configr.AddSource(configr.NewFileSource("/tmp/config.json"))
```

Parse your config:
```go
	if err := configr.Parse(); err != nil {
		...
	}
```

And use at your own leisure:
```go
	fromAddress, err := configr.String("email.fromAddress")
	if err != nil {
		...
	}
```

More examples can be found in the `examples/` dir.

## TODO:
- Concurrent safety, particularly in multi `Parse()`'ing systems and when adding sources (will allow for hot reloads)
- FileSource needs to be refactored to reduce dependency needs, something similar to sql package with a central register and blank importing the flavour you need
- More available sources, Env vars, Flags... etc
