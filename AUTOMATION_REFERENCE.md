# Blockly Tool Automation Reference

This document contains examples and formats for programmatically creating Adafruit IO Actions using the Blockly Tool.

## Test Case: Liquid Template with Sunset Countdown

This example demonstrates a working Liquid template integration that calculates time until sunset.

### Workspace JSON Format

The workspace JSON format represents the visual block structure in the Blockly editor:

```json
{
  "blocks": {
    "languageVersion": 0,
    "blocks": [
      {
        "type": "action_root",
        "movable": false,
        "deletable": false,
        "x": 50,
        "y": 50,
        "extraState": {
          "delaySeconds": 0,
          "delayMode": "extend"
        },
        "inputs": {
          "TRIGGERS": {},
          "EXPRESSIONS": {
            "block": {
              "type": "io_variables_set",
              "fields": {
                "VAR": {
                  "id": "io_variables:weather"
                }
              },
              "inputs": {
                "VALUE": {
                  "block": {
                    "type": "io_text",
                    "fields": {
                      "TEXT": ""
                    }
                  },
                  "shadow": {
                    "type": "io_text"
                  }
                }
              },
              "next": {
                "block": {
                  "type": "io_variables_set",
                  "fields": {
                    "VAR": {
                      "id": "io_variables:sunset_in"
                    }
                  },
                  "inputs": {
                    "VALUE": {
                      "block": {
                        "type": "text_template",
                        "inputs": {
                          "TEMPLATE": {
                            "block": {
                              "type": "io_text_multiline",
                              "fields": {
                                "TEXT": "{%- comment -%} Calculate sunset time difference {%- endcomment -%}\n{%- assign now_timestamp = \"now\" | date: \"%Y-%m-%dT%H:%M:%S%z\" | date: \"%s\" -%}\n{%- assign sunset_timestamp = \"2025-06-30T21:19:30Z\" | date: \"%s\" -%}\n{%- assign time_diff = sunset_timestamp | minus: now_timestamp -%}\n{%- if time_diff > 0 -%}\n  {%- assign hours = time_diff | divided_by: 3600 -%}\n  {%- assign remaining_seconds = time_diff | modulo: 3600 -%}\n  {%- assign minutes = remaining_seconds | divided_by: 60 -%}\n  {%- if hours > 0 -%}\n    {%- if minutes > 0 -%}\n      {{ hours }} hours and {{ minutes }} minutes\n    {%- else -%}\n      {{ hours }} hours\n    {%- endif -%}\n  {%- else -%}\n    {{ minutes }} minutes\n  {%- endif -%}\n{%- else -%}\nthe past (sunset has already occurred)\n{%- endif -%}"
                              }
                            },
                            "shadow": {
                              "type": "io_text"
                            }
                          }
                        }
                      },
                      "shadow": {
                        "type": "io_text"
                      }
                    }
                  },
                  "next": {
                    "block": {
                      "type": "action_log",
                      "inputs": {
                        "EXPRESSION": {
                          "block": {
                            "type": "io_variables_get",
                            "fields": {
                              "VAR": {
                                "id": "io_variables:sunset_in"
                              }
                            }
                          },
                          "shadow": {
                            "type": "io_text"
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    ]
  },
  "variables": [
    {
      "name": "weather",
      "id": "io_variables:weather"
    },
    {
      "name": "sunset_in",
      "id": "io_variables:sunset_in"
    }
  ],
  "procedures": []
}
```

### Bytecode JSON Format

The bytecode format is the compiled/executable representation of the action:

```json
{
  "version": "1.0.0-beta.1",
  "settings": {},
  "triggers": [],
  "expressions": [
    {
      "setVariable": {
        "name": "weather",
        "value": ""
      }
    },
    {
      "setVariable": {
        "name": "sunset_in",
        "value": {
          "textTemplate": {
            "template": "{%- comment -%} Calculate sunset time difference {%- endcomment -%}\n{%- assign now_timestamp = \"now\" | date: \"%Y-%m-%dT%H:%M:%S%z\" | date: \"%s\" -%}\n{%- assign sunset_timestamp = \"2025-06-30T21:19:30Z\" | date: \"%s\" -%}\n{%- assign time_diff = sunset_timestamp | minus: now_timestamp -%}\n{%- if time_diff > 0 -%}\n  {%- assign hours = time_diff | divided_by: 3600 -%}\n  {%- assign remaining_seconds = time_diff | modulo: 3600 -%}\n  {%- assign minutes = remaining_seconds | divided_by: 60 -%}\n  {%- if hours > 0 -%}\n    {%- if minutes > 0 -%}\n      {{ hours }} hours and {{ minutes }} minutes\n    {%- else -%}\n      {{ hours }} hours\n    {%- endif -%}\n  {%- else -%}\n    {{ minutes }} minutes\n  {%- endif -%}\n{%- else -%}\nthe past (sunset has already occurred)\n{%- endif -%}"
          }
        }
      }
    },
    {
      "logAction": {
        "line": {
          "getVariable": {
            "name": "sunset_in"
          }
        }
      }
    }
  ]
}
```

## Key Block Types for StateSmith Integration

### text_template Block
- **Type**: `text_template`
- **Purpose**: Renders Liquid templates
- **Key Field**: `TEMPLATE` input containing the Liquid template text
- **Usage**: Perfect for StateSmith-generated Liquid templates

### Variable Operations
- **Set Variable**: `io_variables_set`
- **Get Variable**: `io_variables_get` 
- **Variable Declaration**: Added to `variables` array with `name` and `id`

### Action Blocks
- **Log Action**: `action_log` - Outputs to action logs
- **Root Action**: `action_root` - Container for the entire action

## Automation Use Cases

1. **StateSmith → Blockly Workflow**:
   - Generate `.liquid` templates from StateSmith state machines
   - Programmatically create workspace JSON with `text_template` blocks
   - Convert to bytecode for Adafruit IO deployment

2. **Template Variables**:
   - Use `feeds['feed-name']` syntax for Adafruit IO feeds
   - State machine variables as Liquid `{{ variable }}` syntax
   - Control flow with `{% if %}`, `{% for %}`, etc.

3. **Integration Points**:
   - StateSmith Liquid transpiler → Template files
   - Template files → Blockly workspace JSON
   - Workspace JSON → Bytecode for execution

## Playwright Automation Guide

### Essential Drag-and-Drop Patterns

#### 1. Workspace Preparation
```javascript
// Always clear workspace first to avoid element conflicts
await page.getByRole('button', { name: 'Clear' }).click();
```

#### 2. Toolbox Navigation
```javascript
// Click specific category to open flyout
await page.getByRole('treeitem', { name: 'Utility' }).click();
await page.getByRole('treeitem', { name: 'Text' }).click();
```

#### 3. Block Connections

**Vertical Chaining (Multiple Expressions)**:
```javascript
// First: Connect log action to Actions section
await page.getByText('" Log:').dragTo(page.getByText('Actions:'));

// Second: Chain next log action to bottom of first (creates "next" connection)
await page.getByText('" Log:').nth(1).dragTo(page.getByText('" Log:').first());
```

**Horizontal Input Connections**:
```javascript
// Connect template to log action text input
await page.getByText('P {{').dragTo(page.locator('.blocklyDraggable > g > g > .blocklyText').first());
```

#### 4. Critical Success Patterns

**Flyout Management**:
- Flyouts auto-close after each drag operation
- Always reclick toolbox category before next drag
- Use nth() selectors for multiple similar elements

**Connection Types**:
- **Vertical (bumps/nubs)**: Action/statement blocks chain vertically → Creates `"next"` connections → Multiple expressions in array
- **Horizontal (puzzle pieces)**: Value/expression blocks connect horizontally → Creates input connections → Nested block structure

**Block Categories by Connection Type**:
- **Action Blocks** (vertical): `action_log`, `action_email`, `io_variables_set` - Chain together with bumps/nubs
- **Value Blocks** (horizontal): `text_template`, `io_text`, `io_variables_get`, logic conditionals - Connect with puzzle pieces
- **Container Blocks**: `action_root` - Has connection slots for both types

**Element Targeting**:
- Target connection areas, not text content
- Use specific selectors like `.blocklyDraggable > g > g > .blocklyText`
- Avoid overlapping elements that cause timeouts

#### 5. Automation Sequence Template

```javascript
// 1. Clear workspace
await page.getByRole('button', { name: 'Clear' }).click();

// 2. Add first log action
await page.getByRole('treeitem', { name: 'Utility' }).click();
await page.getByText('" Log:').dragTo(page.getByText('Actions:'));

// 3. Add second log action (vertical chain)
await page.getByRole('treeitem', { name: 'Utility' }).click();
await page.getByText('" Log:').nth(1).dragTo(page.getByText('" Log:').first());

// 4. Add template to first log
await page.getByRole('treeitem', { name: 'Text' }).click();
await page.getByText('P {{').dragTo(page.locator('.blocklyDraggable > g > g > .blocklyText').first());

// 5. Add template to second log
await page.getByRole('treeitem', { name: 'Text' }).click();
await page.getByText('P {{').nth(1).dragTo(page.locator('.blocklyDraggable > g > g > .blocklyText').first());
```

#### 6. Expected Bytecode Structure

**Success Indicators**:
```json
{
  "expressions": [
    {
      "logAction": {
        "line": {
          "textTemplate": {
            "template": ""
          }
        }
      }
    },
    {
      "logAction": {
        "line": {
          "textTemplate": {
            "template": ""
          }
        }
      }
    }
  ]
}
```

**Workspace JSON Indicators**:
- `"next"` property in first block indicates vertical chaining
- `"textTemplate"` type in EXPRESSION inputs indicates successful template connection

#### 7. Common Pitfalls to Avoid

❌ **Element Overlap Timeouts**: Don't target text elements when blocks overlap
❌ **Closed Flyouts**: Don't assume flyouts stay open between operations  
❌ **Wrong Connection Types**: Vertical for chaining, horizontal for inputs
❌ **Dirty Workspace**: Always clear first to avoid element ID conflicts

✅ **Success Indicators**: Multiple objects in expressions array, textTemplate structures present

## Test Status ✅

- **Liquid Template Parsing**: Confirmed working
- **Complex Liquid Syntax**: Date calculations, conditionals, loops all supported
- **StateSmith Integration**: Liquid transpiler generates compatible templates
- **Feed Integration**: `feeds['feed-key']` syntax supported
- **Playwright Automation**: Drag-and-drop patterns documented and tested

This confirms that StateSmith's Liquid template output is fully compatible with the Blockly Tool and can be used for automated action creation.