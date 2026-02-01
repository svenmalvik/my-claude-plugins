---
name: bubbletea-tui
description: Use when building terminal user interfaces with Bubble Tea, implementing interactive TUI components, handling keyboard input, or managing TUI state. Covers the Elm architecture, common components, and Lip Gloss styling.
---

# Bubble Tea TUI Development

## Core Concepts

Bubble Tea uses the Elm architecture: **Model → Update → View**

```go
type model struct {
    cursor   int
    choices  []string
    selected map[int]struct{}
}

func (m model) Init() tea.Cmd {
    return nil  // Initial command (or tea.Batch for multiple)
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "q", "ctrl+c":
            return m, tea.Quit
        case "up", "k":
            if m.cursor > 0 {
                m.cursor--
            }
        case "down", "j":
            if m.cursor < len(m.choices)-1 {
                m.cursor++
            }
        case "enter", " ":
            if _, ok := m.selected[m.cursor]; ok {
                delete(m.selected, m.cursor)
            } else {
                m.selected[m.cursor] = struct{}{}
            }
        }
    }
    return m, nil
}

func (m model) View() string {
    s := "Select items:\n\n"
    for i, choice := range m.choices {
        cursor := " "
        if m.cursor == i {
            cursor = ">"
        }
        checked := " "
        if _, ok := m.selected[i]; ok {
            checked = "x"
        }
        s += fmt.Sprintf("%s [%s] %s\n", cursor, checked, choice)
    }
    s += "\nPress q to quit.\n"
    return s
}
```

## Running the Program

```go
func main() {
    m := model{
        choices:  []string{"Option 1", "Option 2", "Option 3"},
        selected: make(map[int]struct{}),
    }
    p := tea.NewProgram(m)
    if _, err := p.Run(); err != nil {
        log.Fatal(err)
    }
}

// With options
p := tea.NewProgram(m,
    tea.WithAltScreen(),       // Full-screen mode
    tea.WithMouseCellMotion(), // Mouse support
)
```

## Common Messages

```go
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        // Keyboard input
        switch msg.String() {
        case "ctrl+c": return m, tea.Quit
        }

    case tea.WindowSizeMsg:
        // Terminal resized
        m.width = msg.Width
        m.height = msg.Height

    case tea.MouseMsg:
        // Mouse events (if enabled)
        if msg.Action == tea.MouseActionPress {
            m.cursor = msg.Y
        }

    case tickMsg:
        // Custom message from Cmd
        return m, tick()
    }
    return m, nil
}
```

## Commands and Async

```go
// Tick command for animations
type tickMsg time.Time

func tick() tea.Cmd {
    return tea.Tick(time.Second, func(t time.Time) tea.Msg {
        return tickMsg(t)
    })
}

// Async operation
type fetchResultMsg struct {
    data string
    err  error
}

func fetchData(url string) tea.Cmd {
    return func() tea.Msg {
        resp, err := http.Get(url)
        if err != nil {
            return fetchResultMsg{err: err}
        }
        defer resp.Body.Close()
        data, _ := io.ReadAll(resp.Body)
        return fetchResultMsg{data: string(data)}
    }
}

// Batch multiple commands
func (m model) Init() tea.Cmd {
    return tea.Batch(
        tick(),
        fetchData("https://api.example.com"),
    )
}
```

## Bubbles Components

```go
import (
    "github.com/charmbracelet/bubbles/textinput"
    "github.com/charmbracelet/bubbles/spinner"
    "github.com/charmbracelet/bubbles/list"
)

// Text input
type model struct {
    input textinput.Model
}

func newModel() model {
    ti := textinput.New()
    ti.Placeholder = "Enter name..."
    ti.Focus()
    return model{input: ti}
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    var cmd tea.Cmd
    m.input, cmd = m.input.Update(msg)
    return m, cmd
}

func (m model) View() string {
    return m.input.View()
}

// Spinner
type model struct {
    spinner spinner.Model
    loading bool
}

func newModel() model {
    s := spinner.New()
    s.Spinner = spinner.Dot
    return model{spinner: s, loading: true}
}
```

## Lip Gloss Styling

```go
import "github.com/charmbracelet/lipgloss"

var (
    titleStyle = lipgloss.NewStyle().
        Bold(true).
        Foreground(lipgloss.Color("205")).
        MarginBottom(1)

    selectedStyle = lipgloss.NewStyle().
        Foreground(lipgloss.Color("86")).
        Bold(true)

    boxStyle = lipgloss.NewStyle().
        Border(lipgloss.RoundedBorder()).
        BorderForeground(lipgloss.Color("62")).
        Padding(1, 2)
)

func (m model) View() string {
    title := titleStyle.Render("My App")
    content := boxStyle.Render(m.content)
    return lipgloss.JoinVertical(lipgloss.Left, title, content)
}

// Layout
lipgloss.JoinHorizontal(lipgloss.Top, left, right)
lipgloss.JoinVertical(lipgloss.Center, top, bottom)
lipgloss.Place(width, height, lipgloss.Center, lipgloss.Center, content)
```

## State Management Patterns

```go
// View states
type state int

const (
    stateLoading state = iota
    stateReady
    stateError
)

type model struct {
    state   state
    data    []Item
    err     error
    spinner spinner.Model
}

func (m model) View() string {
    switch m.state {
    case stateLoading:
        return m.spinner.View() + " Loading..."
    case stateError:
        return "Error: " + m.err.Error()
    default:
        return m.renderData()
    }
}

// Nested models for complex UIs
type model struct {
    tabs     []string
    activeTab int
    content  []tea.Model  // Different model per tab
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    // Update active tab's model
    var cmd tea.Cmd
    m.content[m.activeTab], cmd = m.content[m.activeTab].Update(msg)
    return m, cmd
}
```

## Quick Reference

| Pattern | Code |
|---------|------|
| Quit | `return m, tea.Quit` |
| No-op | `return m, nil` |
| Clear screen | `return m, tea.ClearScreen` |
| Batch commands | `tea.Batch(cmd1, cmd2)` |
| Sequence commands | `tea.Sequence(cmd1, cmd2)` |
| Get terminal size | Handle `tea.WindowSizeMsg` |
| Focus component | `m.input.Focus()` |
| Blur component | `m.input.Blur()` |

## Progress and Viewport

```go
import (
    "github.com/charmbracelet/bubbles/progress"
    "github.com/charmbracelet/bubbles/viewport"
)

// Progress bar
type model struct {
    progress progress.Model
    percent  float64
}

func newModel() model {
    return model{
        progress: progress.New(progress.WithDefaultGradient()),
    }
}

func (m model) View() string {
    return m.progress.ViewAs(m.percent)
}

// Viewport for scrolling content
type model struct {
    viewport viewport.Model
    content  string
    ready    bool
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        if !m.ready {
            m.viewport = viewport.New(msg.Width, msg.Height-4)
            m.viewport.SetContent(m.content)
            m.ready = true
        } else {
            m.viewport.Width = msg.Width
            m.viewport.Height = msg.Height - 4
        }
    }
    var cmd tea.Cmd
    m.viewport, cmd = m.viewport.Update(msg)
    return m, cmd
}
```

## Key Bindings with Bubbles/Key

```go
import "github.com/charmbracelet/bubbles/key"

type keyMap struct {
    Up    key.Binding
    Down  key.Binding
    Help  key.Binding
    Quit  key.Binding
}

var keys = keyMap{
    Up: key.NewBinding(
        key.WithKeys("up", "k"),
        key.WithHelp("↑/k", "move up"),
    ),
    Down: key.NewBinding(
        key.WithKeys("down", "j"),
        key.WithHelp("↓/j", "move down"),
    ),
    Help: key.NewBinding(
        key.WithKeys("?"),
        key.WithHelp("?", "toggle help"),
    ),
    Quit: key.NewBinding(
        key.WithKeys("q", "ctrl+c"),
        key.WithHelp("q", "quit"),
    ),
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch {
        case key.Matches(msg, keys.Up):
            m.cursor--
        case key.Matches(msg, keys.Down):
            m.cursor++
        case key.Matches(msg, keys.Quit):
            return m, tea.Quit
        }
    }
    return m, nil
}
```

## Help Component

```go
import "github.com/charmbracelet/bubbles/help"

type model struct {
    help     help.Model
    keys     keyMap
    showHelp bool
}

func (k keyMap) ShortHelp() []key.Binding {
    return []key.Binding{k.Help, k.Quit}
}

func (k keyMap) FullHelp() [][]key.Binding {
    return [][]key.Binding{
        {k.Up, k.Down},
        {k.Help, k.Quit},
    }
}

func (m model) View() string {
    helpView := m.help.View(m.keys)
    return m.content + "\n" + helpView
}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Mutating model directly | Return new model from Update |
| Blocking in Update | Use Cmd for async work |
| View returns empty | Always return a string, even "" |
| No quit handler | Handle `ctrl+c` or `q` |
| Ignoring WindowSizeMsg | Store and use for responsive layout |
| Not initializing bubbles | Call `New()` and set properties |
| Hardcoded dimensions | Use WindowSizeMsg for responsive UI |
| No loading state | Show spinner during async operations |

## Integration with Cobra

```go
var tuiCmd = &cobra.Command{
    Use:   "tui",
    Short: "Launch interactive mode",
    RunE: func(cmd *cobra.Command, args []string) error {
        m := newModel()
        p := tea.NewProgram(m, tea.WithAltScreen())
        finalModel, err := p.Run()
        if err != nil {
            return err
        }
        // Access final state
        if fm, ok := finalModel.(model); ok {
            fmt.Println("Selected:", fm.selected)
        }
        return nil
    },
}
```
