# FOON-to-PDDL (FOON\_to\_PDDL.py) #

This code repository contains Python scripts that are designed to convert files from the [**FOON**](http://www.foonets.com) (short for the **functional object-oriented network**) dataset into [**PDDL**](https://planning.wiki/) problem and domain files.

This requires code (specifically the ```FOON_graph_analyzer.py```, ```FOON_retrieval.py```, and ```FOON_classes.py``` files) from the **FOON\_API** repository, which can be found in this repository under the **FOON_scripts** folder.

<!---
which can be found [here](https://bitbucket.org/davidpaulius/foon_api/src/master/).
-->

## License

>    This program is free software: you can redistribute it and/or modify
>    it under the terms of the GNU General Public License as published by
>    the Free Software Foundation, either version 3 of the License, or
>    (at your option) any later version.
>
>    This program is distributed in the hope that it will be useful,
>    but WITHOUT ANY WARRANTY; without even the implied warranty of
>    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
>    GNU General Public License for more details.
>
>    You should have received a copy of the GNU General Public License
>    along with this program.  If not, see <https://www.gnu.org/licenses/>.


## Running the FOON\_to\_PDDL.py script

To run this code (using Python 3), simply use the following line in your terminal or command line:
```
>> python FOON_to_PDDL.py --file='example.txt' [ --type=1/2] [--help]
```

Where ```example.txt``` in ```--file'example.txt'``` is the name of the text file containing the FOON graph description. 

There is an optional parameter ```--type```, which is used to only produce a single file (either domain or problem). The parameter ```--type``` takes a value of either ```1``` (domain) or ```2``` (problem). 



## What is happening under the hood?

A FOON subgraph file describes a cooking procedure in the form of a bipartite graph (meaning it has two types of nodes).
The _FOON graph analyzer_ (or _FGA_) (found in the **FOON_API** repository) is used to load a graph from a subgraph file.

### Translating a FOON graph to a FOON domain file

From a loaded subgraph, each functional unit is translated into _planning operators_ (defined as ```:action```). The _input nodes_ are used to define the preconditions (defined as ```:preconditions```), and the _output nodes_ are used to define the effects of the action (defined as ```:effects```). 

This is done with the following steps:

1. Take the name of an object node and set that to the name of the current object in focus (denoted as ```<focus_object>```).

2. Parse through all of the states of the object node, taking note of the following:

	-- If a node has some spatial/geometric relation state to another object (e.g. ```in [bowl]```), then the relation and the relative object are taken to produce the predicate (e.g. ```(in bowl <focus_object>)```). This is done for relations ```in```, ```on```, and ```under```. Please refer to [Agostini et al.](https://arxiv.org/abs/2007.08251) for more details on object-centered predicates.
	
	-- Currently, the object-centered predicate relations such as ```left``` and ```right``` are not considered.

	-- If a node has a physical state that cannot be described with object-centered predicates, but which is relevant to the action (based on one's requirements), then create a predicate for that state. Examples of such states are ```whole```, ```chopped```, and ```mixed```. Many others exist in FOON, and these are based on states as discussed in [Jelodar et al.](https://arxiv.org/abs/1805.06956).
	
	-- Other states can be added by simply making modifications to the ```FOON_to_PDDL.py``` script and adding new state terms for parsing.

3. If there is no indication of a spatial/geometric relation state, then assume that the object is on the working surface and the surface is under the object (i.e., ```(on table <focus_object>)``` and ```(under <focus_object> table)```).

Objects were assumed to be constants (i.e., only one instance of each object), but multiple instances of objects could be considered. However, this is not native to FOON. Therefore, further modifications would be required for problems such as object grounding.

### Translating a FOON graph to a FOON problem file
The ```:init``` section of the problem file considers all _starting nodes_ in the FOON file. *Starting nodes* are those nodes that are never seen as output nodes. This carries the assumption that these objects are in their _basic or natural_ state. All of these nodes are identified using a function from the FGA (```fga._identifyKitchenItems()```), which simply uses a dictionary built from the entire graph to identify such nodes. 

For the translation of each node, the same rules as above are applied to create appropriate predicates.



## Using the FOON domain and problem files

Once the files have been generated, you can use any off-the-shelf planner (e.g., [PDDL4J](https://github.com/pellierd/pddl4j) or [Fast-Downward](https://github.com/aibasel/downward)) to see if a plan can be generated. If a plan cannot be found with any FOON graph file that you are testing, be sure to carefully review the problem file for any rogue predicates that are not being satisfied in planning.

For example, with Fast-Downward, you can use the following command:
```
>> python path/to/fast-downward.py --alias seq-opt-lmcut <name_of_domain_file>.pddl <name_of_problem_file>.pddl
```

You can read more about what the ```--alias``` argument flag means [here](https://www.fast-downward.org/IpcPlanners). For now, just know that it is one type of searching approach that is available in the Fast-Downward planner.


## FOON Graphs for Translation

There are two examples provided in this repository: ```FOON-pour_water.txt``` (a very, very basic example of pouring water from a cup to a bowl) and ```FOON-0076-bloody_mary.txt``` (which is a simplified version of the version found in the FOON dataset).

Other graphs can also be downloaded from the **FOON\_API** repository or the [FOON website](http://foonets.com/foon_subgraphs/subgraphs/). It is much easier to start with a regular FOON file and then edit it rather than writing one from scratch due to the precise formatting required.


### Visualizing FOON Graphs

<img src="https://user-images.githubusercontent.com/11097628/145078748-1429b4f1-6300-43fa-a4f1-14a18885ae63.png" alt="drawing" width="400"/>

If you would like to visualize graphs (such as above), you can use the [FOON_view](https://github.com/davidpaulius/foon_view) tool. You can also access the tool directly through the FOON website [here](http://foonets.com/FOON_view/visualizer.html).


## Need Assistance? Have Questions about Papers?

Please contact the main developer David Paulius at <davidpaulius@usf.edu> or <dpaulius@cs.brown.edu>.


