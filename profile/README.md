# luvml

**Type-safe, fluent, composable Java DSL for HTML, CSS, and Vue.js templates.**

Java code that mirrors the HTML/CSS it produces — with compile-time safety, IDE refactoring, and zero template syntax errors.

```java
var page = html(
    head(title("My App"), style(appStyles())),
    body(
        div(id("app"), class_(container),
            h1("Welcome"),
            p("Type-safe HTML in Java.")
        )
    )
);
```

## Libraries

| Library | Purpose | Maven |
|---------|---------|-------|
| **luvml** | HTML/XHTML generation | `io.github.luvml:luvml:2.0` |
| **luvs** | CSS generation (394 properties) | `io.github.luvml:luvs:2.0` |
| **luvue** | Vue.js directive integration | `io.github.luvml:luvue:2.0` |

Adopt incrementally — start with luvml, add the others when you need them.

**Requires:** JDK 21+

## Quick Start

```xml
<dependency>
    <groupId>io.github.luvml</groupId>
    <artifactId>luvml</artifactId>
    <version>2.0</version>
</dependency>
```

```java
import static luvml.E.*;
import static luvml.A.*;

var page = html(
    head(meta(charset("UTF-8")), title("Hello")),
    body(
        div(id("app"),
            h1("Hello World"),
            p("Built with luvml.")
        )
    )
);

System.out.println(XHtmlStringRenderer.asFormatted(page, "  "));
```

## Why?

**CSS class names are enum constants:**
```java
enum Styles implements CssClass { container, card, btn; }

// Rename with IDE → every usage updates
// Misspell → compiler error
// Find all references → works
div(class_(container, card), text("Type-safe"))
```

**CSS lives with your components:**
```java
enum Card implements CssClass {
    card, card_header, card_body;

    public static Frag_I<?> card(String title, Frag_I<?>... content) {
        return div(class_(card),
            div(class_(card_header), text(title)),
            div(class_(card_body), frags(content))
        );
    }

    public static CssRules styles() {
        return rules(
            card.____(background(WHITE), border_radius(px(8)),
                      box_shadow(ZERO, px(2), px(8), rgba(0, 0, 0, 0.1))),
            card_header.____(padding(rem(1)), font_weight(BOLD)),
            card_body.____(padding(rem(1)))
        );
    }
}
```

**Vue.js without leaving Java:**
```java
import static luvue.VA.*;

ul(
    li(vFor("todo in todos"), v$key("todo.id"),
        v$class("{ done: todo.completed }"),
        input(typeCheckbox(), vModel("todo.completed")),
        E.span(vText("todo.text"))
    )
)
```

## Complete Example

```java
import static luvml.E.*;
import static luvml.A.*;
import static luvml.T.text;
import static luvs.P.*;
import static luvs.V.*;
import static luvs.HtmlTag.*;
import static luvs.CssRules.*;

public class TodoApp {
    enum Cls implements CssClass { app, todo_list, todo_item, completed, add_btn; }
    enum Theme implements CssVariable { bg_color, text_color, accent; }

    static CssRules styles() {
        return rules(
            $root.____(bg_color.def("#f5f5f5"), text_color.def("#333"), accent.def("#4a90d9")),
            $all.____(margin(ZERO), padding(ZERO), box_sizing(BORDER_BOX)),
            body.____(font_family("system-ui", "sans-serif"),
                      background_color(bg_color.ref()), color(text_color.ref())),
            app.____(max_width(px(600)), margin(rem(2), AUTO)),
            todo_list.____(display(FLEX), flex_direction(COLUMN), gap(rem(0.5))),
            todo_item.____(
                display(FLEX), align_items(AI_CENTER), padding(rem(1)),
                background(WHITE), border_radius(px(8)),
                box_shadow(ZERO, px(1), px(3), rgba(0, 0, 0, 0.1)),
                transition(TRANSFORM, s(0.2))
            ),
            todo_item.hover().____(transform(translateY(px(-2)))),
            completed.____(P.opacity(0.6), text_decoration("line-through")),
            add_btn.____(
                background(accent.ref()), color(WHITE), border("none"),
                padding(rem(0.75), rem(1.5)), border_radius(px(8)),
                cursor(POINTER), transition(BACKGROUND, s(0.3))
            ),
            add_btn.hover().____(P.opacity(0.9)),
            add_btn.disabled().____(P.opacity(0.5), cursor(NOT_ALLOWED))
        );
    }

    public static void main(String[] args) {
        var page = html(
            head(title("Todo App"), style(styles())),
            body(div(class_(app),
                h1("My Todos"),
                div(class_(todo_list),
                    div(class_(todo_item), text("Learn luvml")),
                    div(class_(todo_item, completed), text("Set up project"))
                ),
                luvml.E.button(class_(add_btn), text("Add Todo"))
            ))
        );
        System.out.println("<!DOCTYPE html>\n" + XHtmlStringRenderer.asFormatted(page, "  "));
    }
}
```

## CSS Features

luvs covers 394 CSS properties including modern features:

```java
// CSS variables as typed enums
enum Colors implements CssVariable { primary, accent; }
$root.____(primary.def("#007bff"), accent.def("#28a745"));
color(primary.ref())  // → var(--primary)

// Fluent selectors
nav.descendant(a.hover()).____(...)           // nav a:hover { ... }
btn.and(active).____(...)                     // .btn.active { ... }
video_chip.child(input.typeCheckbox()).____(...)  // .video_chip > input[type="checkbox"] { ... }

// Media queries
media(prefersColorScheme(DARK),
    $root.____(primary.def("#90caf9"))
);

// Container queries, cascade layers, @supports, @font-face
// Keyframes, transitions, transforms, filters
// Calc expressions, color-mix, logical properties, subgrid
```

## Documentation

- [luvml tutorial](prp/luvml_tutorial.md) — HTML generation
- [luvs tutorial](prp/luvs_tutorial.md) — CSS generation
- [luvue README](prp/luvue_README.md) — Vue.js integration
- [Component patterns guide](prp/component-patterns-guide.md) — Architecture patterns

## License

[See LICENSE](LICENSE)
