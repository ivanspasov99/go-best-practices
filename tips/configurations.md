## `envconfig` tags

In Go, you can assign environment variables to structure fields as you can do for JSON, YAML, and others.

`envconfig` tags could define constraints as follows
```go
`envconfig:""`                          //required variable with no default value
`envconfig:optional`                    //optional variable with no default value
`envconfig:default=tech-docs,optional`  //optional variable with default value if a value is not provided
`envconfig:"default=index"`             //required variable with default value if value is not provided
```

### Code example:

In `config.go` file:
```go
package config

import (
	"github.com/vrischmann/envconfig"
)

var appConfig Config // private variable exposed by the AppConfig() function

type Config struct {
	Mongo struct {
		UserName string `envconfig:""`
		Password string `envconfig:""`
		DBUrl    string `envconfig:""`
	}
}

func InitConfig() error {
	appConfig = Config{}
	err := envconfig.Init(&appConfig) /* Init reads the environment variables and populates appConfig object. It searches only for MONGO_USERNAME, MONGO_PASSWORD and MONGO_DBURL because and all it's variations like mongo_username, Mongo_Username, etc */
	return err
}

func AppConfig() Config {
	return appConfig
}
```

In `main.go` file:
```go
package main
func main() {
	err := config.InitConfig() // Initializes the Config object
	if err != nil {
		fmt.Println(err)
	}	
}
```

### How to consume:
```go
configurations := config.AppConfig() // returns an instance of Config object with already populated env variables
url = "mongodb://" + configurations.Mongo.UserName + ":" + configurations.Mongo.Password + "@" + configurations.Mongo.DBUrl
```
