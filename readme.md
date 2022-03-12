# Selection popup

This is the selection popup from [neorg](https://github.com/nvim-neorg/neorg).
**The credit for all the code goes to [vhyrro](https://github.com/vhyrro) and [danymat](https://github.com/danymat).**

It is for producing popups like this:
![task_popup](https://user-images.githubusercontent.com/81827001/151705563-63b65da6-a503-4b3f-8955-e6671391d129.png)

## Usage

You use the popup by creating a selection object on which you can use different methods.
You create the object like this:
```lua
local selection = require("selection_popup").new_selection(<buffer_name>, {})
```
Where `<buffer_name>` is the name for the buffer of the selection popup.
In the table you provide keys for destroying the popup and returning to a previous window.
The default value is:
```
{
    destroy="<ESC>",
    go_back="<BS>"
}
```

### Methods
#### `blank`
Blank is a function to insert blank lines.
##### Parameters
###### `count: number`
Allows to insert multiple empty lines.

#### `destroy`
Destroys the selection popup and the buffer and the buffer it occupied.

#### `text`
Renders text in the popup.
##### Parameters
###### `text: string`
The text to display.
###### `highlight: string`
The highlights group in which the text should be highlighted. (optional)

#### `title`
    Renders at title.
##### Parameters
###### `text: string`
    The title to render.

#### `flag`
    Creates a pressable flag.
##### Parameters
###### `flag: string`
    This is the flag that will need to be pressed.
    This is a single character. The capitalization matters.
###### `description: string`
    The description that will be used to describe the flag.
###### `callback: table|function`
    The callback to invoke or configuration options for the flag.
    Note that you can use the `callback` key in the table to provide a function.
##### Configuration
    The default configuration looks like this:
    ```lua
{
    highlights = {
        -- TODO: Change highlight group names
            key = "NeorgSelectionWindowKey",
        description = "NeorgSelectionWindowKeyname",
        delimiter = "NeorgSelectionWindowArrow",
    },
               delimiter = " -> ",
               -- Whether to destroy the selection popup when this flag is pressed
                   destroy = true,
},
    ```

#### `rflag`
    Generates recursive (nested) flags.
##### Parameters
###### `flag: string`
    This is the flag that will need to be pressed.
    This is a single character. The capitalization matters.
###### `description: string`
    The description that will be used to describe the flag.
###### `callback: table|function`
    The callback to invoke after the flag is entered.

#### `push_page`
    Pushes a new page onto the stack, clearing the buffer and starting fresh.

#### `pop_page`
    Pops the page stack, effectively restoring the previous state.

#### `prompt`
    Creates a prompt inside the page.
##### Parameters
###### `text: string`
    This is the text that will be used for the prompt.
###### `callback: table|function`
    The callback to invoke or configuration options for the flag.
#### `concat`
    Concatenates a `callback` function that returns the selection popup to the existing selection popup
##### Parameters
###### `callback: function`
    The function to append

## Examples
    This would be at the beginning of every example:
    ```lua
    local buffer = create_split("buffer_name")

    -- Binds a selection to that buffer
    local selection =
begin_selection(buffer)

    :listener(
            "destroy",
            { "<Esc>" },
            function(self)
            self:destroy()
            end
            )
    :listener("go-back", { "<BS>" }, function(self)
            self:pop_page()
            end)
    ```

### Get prompt input with default text and print it
    ```lua
    selection:prompt("Input", {
            callback = function(text) print(text) end,
            prompt_text="Default Text"
            })
```

### Insert elements at start and end of a table
    ```lua
local function insert_end(selection, tbl)
    local title = "Insert at End"
    return selection:rflag("b", title, {
            destroy = false,
            callback = function()
            selection
            :listener("go-back", { "<BS>" }, function(self)
                    self:pop_page()
                    end)
            :title(title)
            :blank()
            :prompt(title, {
                    callback = function(text)
                    if #text > 0 then
                    table.insert(tbl, text)
                    end
                    end,
                    pop = true,
                    })
            end,
            })

local function insert_start(selection, tbl)
    local title = "Insert at Start"
    return selection:rflag("a", title, {
            destroy = false,
            callback = function()
            selection
            :listener("go-back", { "<BS>" }, function(self)
                    self:pop_page()
                    end)
            :title(title)
            :blank()
            :prompt(title, {
                    callback = function(text)
                    if #text > 0 then
                    table.insert(tbl, 1, text)
                    end
                    end,
                    pop = true,
                    })
            end,
            })
end
local tbl = {}
selection
:title("Insert Elements")
:blank(2)
    :concat(function()
            return insert_start(selection, tbl)
            end)
     :blank()
    :concat(function()
            return insert_end(selection, tbl)
            end)
     :blank()
      :flag("<CR>", "Print Table", {
              callback = function()
              print(vim.inspect(tbl))
              end,
              destroy = false,
              })
```
