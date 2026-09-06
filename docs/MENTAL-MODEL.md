# TODO App — Mental Model

The goal of V1 is to understand the complete flow without hiding it behind abstractions.

Think in this direction:

```text
DATA
  ↓
TYPE / SHAPE
  ↓
COMPONENT
  ↓
LIST
  ↓
FORM
  ↓
STATE
  ↓
VALIDATION
  ↓
DATABASE
```

Do not build every layer at once. Prove that one layer works before adding the next one.

---

## V1 — Static Todo Card

Start with the UI only.

Mental model:

```text
I have some todo data
        ↓
I give that data to TodoCard
        ↓
TodoCard decides how it should look
        ↓
The page renders TodoCard
```

Pseudo code:

```text
TodoCard
  receive title
  receive items

  render card
    render title

    for each item
      render bullet / checkbox
```

Example data shape:

```text
Todo
  id
  title
  items
```

Possible TypeScript shape later:

```text
Todo = {
  id: string
  title: string
  items: string[]
}
```

At this stage, hardcoded data is fine.

```text
page
  create temporaryTodo

  render TodoCard
    pass temporaryTodo.title
    pass temporaryTodo.items
```

### Goal

Be able to answer:

> How does data reach my component and become UI?

---

## V1.1 — Make the Component Data-Driven

Once one hardcoded card renders correctly, stop putting the todo content inside the component.

Bad mental model:

```text
TodoCard knows the todo
```

Better mental model:

```text
Parent owns the data
        ↓
TodoCard receives the data
        ↓
TodoCard renders it
```

Pseudo code:

```text
function TodoCard(todo)
  display todo.title

  for each item in todo.items
    display item
```

The card should mainly care about presentation.

---

## V1.2 — Render Multiple Todos

Next, move from one object to an array.

```text
todos = [
  todo,
  todo,
  todo
]
```

Mental model:

```text
Page owns todos[]
      ↓
Page loops over todos[]
      ↓
Each todo becomes one TodoCard
```

Pseudo code:

```text
for each todo in todos
  render TodoCard(todo)
```

In React this eventually becomes the `.map()` pattern.

### Goal

Understand:

```text
array of data
      ↓
map
      ↓
array of components
```

---

## V1.3 — Add a Form

Now build the input side of the same data shape.

Do not think:

```text
"I need a form"
```

Think:

```text
"I need a way for the user to create a Todo object"
```

If a Todo looks like:

```text
Todo
  title
  items
```

then the form needs to collect enough information to create that structure.

Pseudo code:

```text
TodoForm
  title input
  item input(s)

  on submit
    create newTodo

    newTodo = {
      id
      title
      items
    }
```

Mental model:

```text
INPUTS
  ↓
FORM VALUES
  ↓
TODO OBJECT
```

---

## V1.4 — Connect Form to State

Before the database, prove the complete UI flow in memory.

```text
User fills form
      ↓
Form creates Todo
      ↓
Todo is added to todos[]
      ↓
React renders todos[] again
      ↓
New TodoCard appears
```

Pseudo code:

```text
todos = current todos

onCreateTodo(newTodo)
  todos = todos + newTodo
```

At this point the app works, but refreshing the browser loses the data.

That is expected.

### Important realization

State answers:

> What data exists in the UI right now?

A database answers:

> What data should still exist after the request/browser/app is gone?

---

## V1.5 — Validate the Boundary

Once data can be created by a user, validate it before trusting it.

Mental model:

```text
unknown user input
      ↓
validation
      ↓
trusted Todo data
```

Pseudo code:

```text
receive form data

validate
  title must exist
  title must have valid length
  items must be valid

if invalid
  return errors

if valid
  continue
```

This is where Zod can later fit naturally.

Do not start with Zod just because it exists. First understand what boundary you are validating.

---

## V1.6 — Add Persistence

Only now replace temporary storage with the database.

Mental model:

```text
FORM
  ↓
VALIDATE
  ↓
WRITE TO DATABASE
  ↓
READ FROM DATABASE
  ↓
RENDER TodoCard
```

Pseudo code for create:

```text
createTodo(formData)
  validate formData

  database.todo.create(validData)
```

Pseudo code for read:

```text
getTodos()
  return database.todo.findMany()
```

Page:

```text
page
  todos = getTodos()

  for each todo
    render TodoCard(todo)
```

This is the same flow as before.

The source of the data changed:

```text
BEFORE
hardcoded array → component

THEN
React state → component

NOW
database → component
```

The component does not need to care where the data came from.

---

# V1 Complete Mental Model

```text
                    ┌──────────────┐
                    │     USER     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │     FORM     │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  VALIDATION  │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   DATABASE   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    TODOS[]   │
                    └──────┬───────┘
                           │
                           ▼
                          map
                           │
                           ▼
                    ┌──────────────┐
                    │  TodoCard    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │      UI      │
                    └──────────────┘
```

For updates and deletes, follow the same idea in reverse:

```text
user action
   ↓
identify todo
   ↓
validate operation
   ↓
change database
   ↓
get/render updated data
```

---

# V2 — JSON Content

JSON is not needed for the first version.

Use normal fields while the structure is predictable.

```text
Todo
  id
  title
  items
  completed
```

JSON starts becoming useful if a Todo can contain different kinds of blocks.

Example mental shape:

```text
Todo
  id
  title
  content[]

content block
  type
  data
```

Example:

```text
content = [
  {
    type: checklist
    items: [...]
  },
  {
    type: note
    text: "Remember this"
  },
  {
    type: link
    url: "..."
  }
]
```

Renderer mental model:

```text
for each block in todo.content

  if block.type is checklist
    render Checklist

  if block.type is note
    render Note

  if block.type is link
    render Link
```

That produces a more Notion-like architecture:

```text
Todo
  ↓
content[]
  ↓
inspect block.type
  ↓
choose component
  ↓
render block
```

Save this for V2. First learn the simpler data flow completely.

---

# Development Rule

When stuck, ask one question:

> What is the data, who owns it, and where does it need to go next?

Then trace it:

```text
Where is the data created?
        ↓
What shape does it have?
        ↓
Who validates it?
        ↓
Who stores it?
        ↓
Who reads it?
        ↓
Which component receives it?
        ↓
How does the component render it?
```

If you can answer those questions, you usually know what code needs to be written next.
