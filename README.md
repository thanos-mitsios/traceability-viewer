# traceability-viewer

## Content Table

- [Getting Started](#GettingStarted)
- [Configuration](#Configuration)
  - [traceability_export](#traceability_export)
  - [html_documentation_root](#html_documentation_root)
  - [layered](#layered)
  - [layers](#layers)
  - [item_colors](#item_colors)
  - [link_colors](#link_colors)
  - [backwards_relationships](#backwards_relationships)
  - [visualised_properties](#visualised_properties)
- [Usage](#Usage)


## Getting Started

First, you need to install docker compose: https://docs.docker.com/compose/install/linux/#install-the-plugin-manually

<!-- https://docs.docker.com/engine/reference/commandline/compose_up/ -->

After installation, copy the JSON file that contains the data to the project folder and copy the `.env.example` to `.env`.
```
# copy traceability_export.json file to project folder
cp /path/to/traceability_export.json .
# copy example .env to your .env
cp .env.example .env
```
Lastly, use docker compose to start the application.

```
docker compose up --build --remove-orphans
```

## Configuration

The configuration takes the form of a YAML file. The [config.yml](config.yml) is an example that requires modification.

### traceability_export

> [!IMPORTANT]
> `traceability_export` is a required configuration parameter.

This is the path to the JSON file that contains all the data. The items in the JSON file needs to have a name and targets.

It should be structured as (this is the minimum required):
```
[
    {
        "name" : "<name-of-item>",
        "targets": {
            "<relation-type-1>": [
                "<target-item-name-1>",
                "<target-item-name-2>",
                ...
            ],
            "<relation-type-2>": [
                "<target-item-name-3>",
                "<target-item-name-4>",
                ...
            ],
        }
    },
    ...
]
```

When you use the [traceability-plugin](https://melexis.github.io/sphinx-traceability-extension/index.html), you can [export](https://melexis.github.io/sphinx-traceability-extension/configuration.html#export) the documentation items as a JSON.

### html_documentation_root

> [!NOTE]
> `html_documentation_root` is not a required configuration parameter.

The html root to the documentation of that item. This can also be the path to a local html folder.

This is only workable when you use the exported JSON of the [traceability-plugin](https://melexis.github.io/sphinx-traceability-extension/configuration.html#export-to-json).


### layered

> [!IMPORTANT]
> `layered` is a required configuration parameter.

The boolean option to layer the nodes or not.
- `true`: the layers will be made according to the [layers](#layers) configuration parameter.
- `false`: there are no layers. The nodes will be force directed.


### layers

> [!NOTE]
> `layers` is only required when `layered` is `true`.

The layers can be a _list_ or a _dict_ of [regular expressions](https://docs.python.org/3/library/re.html#regular-expression-syntax).

- _list_: A list is used when there is only **one** regular expressions needed per layer.
- _dict_: A dictionary is used when there are **two** regular expressions needed per layer.

The order of the layers defines the layer's position. The top and bottom layers are respectively to the first and last layer defined in the _list_ or _dict_.


### item_colors

> [!NOTE]
> `item_colors` is not a required configuration parameter, but keep in mind that when it is not specified, all nodes will be black (`others`).

A dictionary where the key is a regular expression and the value is the desired color of the node. The regular expression is used to [match](https://docs.python.org/3/library/re.html#re.match) (the beginning of) the name of a node with that particalor regular expression.
The color can be an existing [color name](https://www.w3schools.com/tags/ref_colornames.asp) or a [code](https://www.w3schools.com/colors/colors_picker.asp) (hex, rgb or hsl).

If `item_colors` is not specified, it automatically adds following code to the configuration file.
```
item_colors:
    others: black
```
If `item_colors` is defined but `others` isn't, it automatically adds `others: black` to `item_colors`.


### link_colors

> [!NOTE]
> `link_colors` is not a required configuration parameter, but keep in mind that when it is not specified, all relationships between nodes will be black (`others`).

A dictionary where the key is the name of relationship type and the value is the desired color of the relationship.
The color can be an existing [color name](https://www.w3schools.com/tags/ref_colornames.asp) or a [code](https://www.w3schools.com/colors/colors_picker.asp) (hex, rgb or hsl).

If `link_colors` is not specified, it automatically adds following code to the configuration file.
```
link_colors:
    others: black
```
If `link_colors` is defined but `others` isn't, it automatically adds `others: black` to `link_colors`.


### backwards_relationships

> [!IMPORTANT]
> `backwards_relationships` is a required configuration parameter.

A dictionary where the key is the backwards relationship and the value the forward relationship. To reduct the amount of links, only the forward relationhsips (values) will be displayed.

### visualised_properties

> [!NOTE]
> `visualised_properties` is not a required configuration parameter.

The properies of the clicked node that need to be visualised in the as info about that node.

If the `visualised_properties` and [`html_documentation_root`](#html_documentation_root) are not defined , it automatically adds following code to the configuration file.

```
visualised_properties:
    - name
```

If [`html_documentation_root`](#html_documentation_root) is defined it automatically adds following code instead of the previous one.

```
visualised_properties:
    - name
    - document
```

If [`html_documentation_root`](#html_documentation_root) and `visualised_properties` are defined but `document` is not in the list, it automatically adds `document` to the end of the list.

If `visualised_properties` is defined but `name` is missing, it automatically adds `name` to the end of the list


## Usage

### Home Page

The home page looks like the following image. It consists of the bottons on top, two input fields respectively for a Cypher query and search for a node and at the bottom the interactions with showed network.

![home overview](images/home_overview.png)

#### Cypher Query

The input field for submitting a cypher query has two tooltips:
![tooltip help](images/tooltip_help.png)
This is a link to the Cypher cheat sheet of Neo4j.

![tooltip query](images/tooltip_query.png)
This tooltip makes you aware of the keywords that are not allowed.

The input field has an autocompletion as shown in this figure:
![autocomplete query](images/autocomplete_query.png)
Suggestions starting with the same letter as the input, will be on top, followed by the suggestions containing it.

#### Search

The search input field has as well autocompletion as shown in this figure:
![autocomplete query](images/autocomplete_search.png)

The suggestions only contain the names of nodes because there is nothing else allowed.

### Group Filter

The group filter are based on the configuration file. If `layers` is given, these unique groups are groups to filter on. When `layers` is nog defined, the groups are based on the groups of the `item_colors`.
The filter button gives a filtring on the specified group (the name of the button), with all their connecting nodes.
An example of such an filter is shown in the following image.

![filter](images/filter.png)

The legend is based on the visible nodes and links. When diffrent nodes or links are added, the legend will be updated.

### Interactions

#### Hide Groups

When clicking a group of the legend, this group will disappear. When clicking again, it reverts to previous.

![hide group](images/hide_group.png)

#### Hide Links

When clicking a link of the legend, this link will disappear. When clicking again, it reverts to previous.

![hide group](images/hide_link.png)


#### Selecting a Node

When clicking a node, this node will be selected and the info will be visible in the top left corner.

![select node](images/select_node.png)

The image shows the info of the selected node. There are two more buttons showed as well. These two buttons pop up when a node is selected. These will be explained in [highlight connecting nodes](####highlight_connecting_nodes) and [Add/show connecting nodes](####add/show_connecting_nodes).

#### Hovering a Node

When hovering a nodes, the name of that node is shown.
In the previous image, the black node at the bottom was hovered by the mouse.

#### HTML link in Info of Node

When `html_documentation_root` is specified, the name of the node will be a link to the documentation of that specific node.

#### Highlight Connected Nodes

When the button with the eye is clicked, the nodes connecting to the selected node will be highlighted. The rest of the nodes get a lower opacity.

![highlight connected nodes](images/highlight_connected_nodes.png)


#### Add/Show Connecting Nodes

When the button **Add/show connecting nodes** is clicked, the nodes connecting to the selected node will be obtained. It will add new nodes or links, if there are any.

Before pushing the button:
![before adding](images/before_adding.png)

After pushing the button:
![add connecting nodes](images/add_connecting_nodes.png)

#### Node labels

The labels/names of the nodes are shown when the checkbox of **Nodes with labels** is active.

![show labels](images/show_labels.png)

#### Zoom buttons

There are three buttons for zooming, respectively zoom in, zoom out and zoom till the network fits the canvas.

![zoom buttons](images/zoom.png)

## Bugs

1. When showing the labels/names of the nodes, there is allways one node that turns black.
1. When first nodes are added to a certain filter group, then go to another filter and back to the first filter, the nodes that were added are still there.
1. When using the input fields, the autocompletion list will always look at the last part of the sentence, so when adding something in the middle the autocompletion will stay the same (focused on the last part).
1. When something in the legend is hidden, and you go to another filter, the legend is hidden at the same place (for example you hide the first element in the legend, the first element is also hidden when you go to another filter even it is another name).



## Features

- Errors could be popping up, you need to close it before you can go further. That way the user can't miss it.
- Faster rendering; open to suggestions
- Instead of disabling the button when submitted, stop the previous request when another submit is done.
- A node that needs a layer but the group is not specified, better to place it on the same hight as the first node that is connected to it.
- Reset button when in a filter.
