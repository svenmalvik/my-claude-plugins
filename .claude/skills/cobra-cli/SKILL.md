---
name: cobra-cli
description: Use when building Go CLI applications with Cobra, adding commands or flags, structuring CLI projects, or implementing command hierarchies. Covers command patterns, flag binding, and CLI UX.
---

# Cobra CLI Development

## Project Structure

```
cmd/
  root.go           # Root command, global flags
  serve.go          # vce serve
  config/
    config.go       # vce config
    set.go          # vce config set
    get.go          # vce config get
main.go             # Entry: cmd.Execute()
```

## Command Definition

```go
var serveCmd = &cobra.Command{
    Use:   "serve [flags]",
    Short: "Start the API server",
    Long:  `Start the API server with the specified configuration.

Listens on the configured port and serves the REST API.`,
    Example: `  vce serve --port 8080
  vce serve --config ./config.yaml`,
    RunE: func(cmd *cobra.Command, args []string) error {
        return runServe(cmd.Context())
    },
}

func init() {
    rootCmd.AddCommand(serveCmd)
    serveCmd.Flags().IntP("port", "p", 8080, "port to listen on")
    serveCmd.Flags().String("config", "", "config file path")
}
```

## Flag Patterns

```go
// Persistent flags (inherited by subcommands)
rootCmd.PersistentFlags().StringP("output", "o", "table", "output format (table|json|yaml)")
rootCmd.PersistentFlags().BoolP("verbose", "v", false, "verbose output")

// Local flags (this command only)
cmd.Flags().String("name", "", "resource name")
cmd.MarkFlagRequired("name")

// Flag groups
cmd.MarkFlagsOneRequired("file", "stdin")
cmd.MarkFlagsMutuallyExclusive("json", "yaml")

// Bind to viper for config file support
viper.BindPFlag("port", serveCmd.Flags().Lookup("port"))
```

## Subcommand Hierarchy

```go
// Parent command (no Run, just groups subcommands)
var configCmd = &cobra.Command{
    Use:   "config",
    Short: "Manage configuration",
}

// Child commands
var configSetCmd = &cobra.Command{
    Use:   "set KEY VALUE",
    Short: "Set a config value",
    Args:  cobra.ExactArgs(2),
    RunE:  runConfigSet,
}

func init() {
    rootCmd.AddCommand(configCmd)
    configCmd.AddCommand(configSetCmd, configGetCmd)
}
```

## Argument Validation

```go
// Built-in validators
Args: cobra.NoArgs,
Args: cobra.ExactArgs(1),
Args: cobra.MinimumNArgs(1),
Args: cobra.MaximumNArgs(3),
Args: cobra.RangeArgs(1, 3),

// Custom validation
Args: func(cmd *cobra.Command, args []string) error {
    if len(args) < 1 {
        return fmt.Errorf("requires cluster name")
    }
    if !isValidCluster(args[0]) {
        return fmt.Errorf("invalid cluster: %s", args[0])
    }
    return nil
},
```

## Output Formatting

```go
func runList(cmd *cobra.Command, args []string) error {
    items, err := fetchItems()
    if err != nil {
        return err
    }

    format, _ := cmd.Flags().GetString("output")
    switch format {
    case "json":
        return json.NewEncoder(os.Stdout).Encode(items)
    case "yaml":
        return yaml.NewEncoder(os.Stdout).Encode(items)
    default:
        return printTable(items)
    }
}

func printTable(items []Item) error {
    w := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)
    fmt.Fprintln(w, "NAME\tSTATUS\tAGE")
    for _, item := range items {
        fmt.Fprintf(w, "%s\t%s\t%s\n", item.Name, item.Status, item.Age)
    }
    return w.Flush()
}
```

## Error Handling

```go
// Use RunE, not Run
RunE: func(cmd *cobra.Command, args []string) error {
    if err := validate(); err != nil {
        return err  // Cobra prints and exits non-zero
    }
    return execute()
},

// Silence usage on runtime errors
rootCmd.SilenceUsage = true  // Don't show usage for runtime errors

// Custom error types for exit codes
type ExitError struct {
    Code int
    Err  error
}

func (e *ExitError) Error() string { return e.Err.Error() }
```

## Context and Cancellation

```go
func main() {
    ctx, cancel := signal.NotifyContext(context.Background(),
        os.Interrupt, syscall.SIGTERM)
    defer cancel()

    if err := rootCmd.ExecuteContext(ctx); err != nil {
        os.Exit(1)
    }
}

// Access in command
RunE: func(cmd *cobra.Command, args []string) error {
    ctx := cmd.Context()
    return longRunningTask(ctx)
},
```

## Completions

```go
// Static completions
cmd.RegisterFlagCompletionFunc("output", func(cmd *cobra.Command, args []string, toComplete string) ([]string, cobra.ShellCompDirective) {
    return []string{"table", "json", "yaml"}, cobra.ShellCompDirectiveNoFileComp
})

// Dynamic completions
cmd.RegisterFlagCompletionFunc("cluster", func(cmd *cobra.Command, args []string, toComplete string) ([]string, cobra.ShellCompDirective) {
    clusters, _ := listClusters()
    return clusters, cobra.ShellCompDirectiveNoFileComp
})
```

## Quick Reference

| Pattern | Code |
|---------|------|
| Required flag | `cmd.MarkFlagRequired("name")` |
| Hidden command | `cmd.Hidden = true` |
| Deprecated flag | `cmd.Flags().MarkDeprecated("old", "use --new")` |
| Persistent pre-run | `PersistentPreRunE: func(...)` |
| Version flag | `rootCmd.Version = "1.0.0"` |

## Viper Configuration

```go
import "github.com/spf13/viper"

func initConfig() {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        home, _ := os.UserHomeDir()
        viper.AddConfigPath(home)
        viper.AddConfigPath(".")
        viper.SetConfigName(".myapp")
        viper.SetConfigType("yaml")
    }

    viper.AutomaticEnv()
    viper.SetEnvPrefix("MYAPP")
    viper.SetEnvKeyReplacer(strings.NewReplacer("-", "_"))

    if err := viper.ReadInConfig(); err == nil {
        fmt.Fprintln(os.Stderr, "Using config:", viper.ConfigFileUsed())
    }
}

// Bind flags to viper
func init() {
    cobra.OnInitialize(initConfig)
    rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file")
    viper.BindPFlag("port", serveCmd.Flags().Lookup("port"))
}

// Priority: flag > env > config file > default
port := viper.GetInt("port")
```

## Testing CLI Commands

```go
func TestServeCommand(t *testing.T) {
    // Capture output
    buf := new(bytes.Buffer)
    rootCmd.SetOut(buf)
    rootCmd.SetErr(buf)

    // Set args
    rootCmd.SetArgs([]string{"serve", "--port", "9090"})

    // Execute
    err := rootCmd.Execute()
    if err != nil {
        t.Fatalf("execute failed: %v", err)
    }

    // Check output
    if !strings.Contains(buf.String(), "listening") {
        t.Error("expected listening message")
    }
}

func TestRequiredFlag(t *testing.T) {
    rootCmd.SetArgs([]string{"create"})  // missing --name
    err := rootCmd.Execute()
    if err == nil {
        t.Error("expected error for missing required flag")
    }
}

// Test with stdin
func TestStdinInput(t *testing.T) {
    rootCmd.SetIn(strings.NewReader("input data\n"))
    rootCmd.SetArgs([]string{"process", "--stdin"})
    // ...
}
```

## Exit Codes

```go
const (
    ExitSuccess         = 0
    ExitError           = 1
    ExitUsageError      = 2
    ExitConfigError     = 78
    ExitPermissionError = 77
)

func main() {
    if err := rootCmd.Execute(); err != nil {
        var exitErr *ExitError
        if errors.As(err, &exitErr) {
            os.Exit(exitErr.Code)
        }
        os.Exit(ExitError)
    }
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `Run` instead of `RunE` | Always use `RunE` for error handling |
| Global variables for flags | Bind flags in `init()`, read in `RunE` |
| No context propagation | Pass `cmd.Context()` to all functions |
| Missing completions | Add for all flags and arguments |
| Verbose help text | Keep Short brief, use Long for details |
| Not testing commands | Use `SetArgs`, `SetOut`, `SetErr` for tests |
| Ignoring viper priority | Flag > env > config > default |
