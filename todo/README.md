# Topaz Todo Sample

## Table of Content
- [Use-Case](#use-case)
  - [Requirements](#requirements)
  - [Scenario](#scenario)
  - [Expected Outcomes](#expected-outcomes)
- [Modeling in Topaz](#modeling-in-topaz)
  - [Model](#model)
  - [Tuples](#tuples)
  - [Assertions](#assertions)

## Use-Case

This is a model for a Todo application.

### Requirements

- There are users, groups, todos
- Users can be members of groups
- The permissions defined are:
  - create-todo: create a todo
  - delete-todo: delete a todo
  - list-todos: list all todos
  - edit-todo: edit a todo
  - view-todo: view a todo
- Every todo has an owner
- A todo can have a viewer, a todo's viewers are whoever has been directly granted the viewer relation 
- A todo can have an editor, a todo's editors are whoever has been directly granted the editor relation
- Members of the "todo-creators" group are allowed create-todo
- Members of the "todo-editors" group are allowed edit-todo on any todo
- Members of the "todo-viewers" group are allowed view-todo on any todo
- Only a todo's owner is allowed delete-todo
- Todo editors are allowed edit-todo
- Todo viewers are allowed view-todo

### Scenario

- Rick is a member of the todo-creators group
- Morty is a member of the todo-creators group
- Beth is a member of the todo-editors group
- Summer is a member of the todo-editors group
- Jerry is a member of the todo-viewers group
- Rick is the owner of rick-todo
- Jerry is the owner of jerry-todo
- Summer is the owner of summer-todo
- Jerry is an editor of summer-todo

### Expected Outcomes

- Rick can delete "rick-todo"
- Rick can edit "rick-todo"
- Morty **cannot** delete "rick-todo"
- Morty can edit "rick-todo"
- Jerry can view "rick-todo"
- Jerry can edit "summer-todo"

## Modeling in Topaz
### Model (manifest.yaml)

```yaml
---
# todo (object type)
todo:
  # todo:owner (object relation)
  owner:
    # incl permissions of other object relations on the same object type
    union: 
    - editor
    - viewer
    permissions:
    - delete-todo

  # todo:editor (object relation)
  editor:
    union: 
    - viewer
    permissions:
    - edit-todo

  # todo:viewer (object relation)
  viewer:
    union:
    permissions:
    - list-todos
    - view-todos
```

The model is represented in this file: [manifest.yaml](./model/manifest.yaml)

### Tuples

| User                  | Relation | Object              | Description                                                            |
|-----------------------|----------|---------------------|------------------------------------------------------------------------|
| rick                  | member   | todo-creators       | Rick is a member of the todo-creators group                                  |
| morty                 | member   | todo-creators       | Morty is a member of the todo-creators group                                  |
| summer                | member   | todo-editors        | Summer is a member of the todo-editors group                              |
| beth                  | member   | todo-editors        | Beth is a member of the todo-editors group                              |
| jerry                 | member   | todo-viewers        | Jerry is a member of the todo-viewers group                              |
| rick                  | owner    | rick-todo           | Rick owns rick-todo           |
| morty                 | owner    | morty-todo          | morty owns morty-todo         |
| summer                | owner    | summer-todo         | Summer owns summer-todo       |
| jerry                 | editor   | summer-todo         | Jerry is an editor of summer-todo       |

These are represented in these files: 

* [objects.json](./data/objects.json)
* [relations.json](./data/relations.json)

### Assertions

| User    | Permission   | Object             | Allowed? |
|---------|--------------|--------------------|----------|
| rick    | delete-todo  | rick-todo          | Yes      |
| rick    | edit-todo    | rick-todo          | Yes      |
| morty   | delete-todo  | rick-todo          | No       |
| morty   | edit-todo    | rick-todo          | Yes      |
| jerry   | view-todo    | rick-todo          | Yes      |
| jerry   | edit-todo    | summer-todo        | Yes      |

These are represented in this file: [assertions.json](./test/assertions.json).

## Validating the model

To validate the model described above:

1. Clone the topaz-samples repo:

	```
	mkdir -p ~/workspace/topaz
	cd ~/workspace/topaz
	git clone https://github.com/aserto-dev/topaz-samples.git
	```

2. Set the current working directory to the `todo` sample directory

	```
	cd ~/workspace/topaz/topaz-samples/todo
	```

3. Install a topaz instance, see [Topaz Getting Started](https://www.topaz.sh/docs/getting-started)

	```
	topaz install
	```

4. Configure the topaz instance, using:
 
	```
	topaz configure gdrive --resource=opcr.io/aserto-templates/policy-template:latest -d -s
	```

5. Start your topaz instace, using:

	```
	topaz start
	```

6. Load the domain model, using the manifest file:

	```
	topaz load ./model/manifest.yaml --insecure
	```

7. Load the sample objects and relations, using:

	```
	topaz import --directory ./data --insecure
	```

8.	Validate the assertions, using:

	```
	./assert.sh
	``` 

9. Validate the results:

	```
	./assert.sh
	PASS
	PASS
	FAIL REQ:{"subject":{"type":"user","key":"morty"},"permission":{"name":"delete-todo"},"object":{"type":"todo","key":"rick-todo"}}
	PASS
	PASS
	PASS
	``` 
