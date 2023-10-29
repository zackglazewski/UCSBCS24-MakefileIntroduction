# CS24 - Introduction to Makefiles

## What is a Makefile?
At a very general level, makefiles allow us to define aliases for commands that we may run on our command line. We might have a really long command that we would like to use and modify over and over again. 

For example, consider the following sequence of commands:

```
$ g++ file1.cpp file1.h -c
$ g++ file2.cpp file2.h -c
$ g++ main.cpp -c
$ g++ main.o file1.o file2.o -o app
```

Here is a depiction of the compilation process for a project which includes file1.cpp/h, file2.cpp/h and main.cpp. Most of the time, when we make a change to one of these files, it means we must *recompile* the project, by calling the above sequence of commands. 

Makefiles give us the power to write an alias for this set of commands. We could configure a makefile such that, only writing something like:
```
$ make app
```

will call all of the commands needed to compile our source code into our executable. "app" in this case, is the makefile alias. 

### A compilation crash course
Throughout this, you may get confused by the compilation commands. Very briefly, I will cover some core ideas.

Compilation refers to the process of taking source code and converting it to something readable to your computer (binary). g++ is a compiler that we often use, along with clang++. The syntax for compiling is generally:

```[compiler] [source files] [flags] [target]```

Sometimes, we break our project down into helper files (like file1.cpp/h and file2.cpp/h)
In cases like these, it may be ideal to compile these to object files and then link them later. 

##### Flags
```-o``` stands for "compile and link". Not only will this compile source files to object files, it will link them together to form an executable you can run. 

```-c``` stands for "compile only". This converts a source file to an object (.o) file, which you can link later. 

### The Recipe Analogy
Your friend is a chef. As a chef, they have a giant recipe book. Each recipe has a recipe name, a list of ingredients, and a step by step list of instructions of how to combine your ingredients in order to produce the recipe. 

Now suppose you asked your friend to make some Cherry Pie!
1. They flip open their recipe book to the recipe whose name is "Cherry Pie"
2. They scan the ingredients and check if they have every single one
3. Once they obtain every ingredient, they follow the instructions, and... now we have a Cherry Pie!

Now back to your computer. A makefile is a recipe book and now your computer is the chef!

Now suppose you ask your computer to make an application called "app"
1. Your computer scans the makefile to search for the recipe whose name is "app"
2. Your friend scans the recipe's dependency list and check if you have every single one
3. Once every dependency is present, you execute the list of commands for that recipe and... you now have an application called "app"!

### The Anatomy of a Makefile
Again, every makefile is a list of recipes. Each recipe has a name, dependencies (ingredients) and commands (instructions) to execute. 

The general syntax for a recipe is as follows:
```Makefile
recipeName: dependencyList
    first command
    ...
    last command
```
In our Cherry Pie example:
```Makefile
CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven
```

And for our "app" example:
```Makefile
app: main.cpp file1.cpp file1.h file2.cpp file2.h
    g++ file1.cpp file1.h -c
    g++ file2.cpp file2.h -c
    g++ main.cpp -c
    g++ main.o file1.o file2.o -o app
```

### The Case of the Missing Pie Crust
For the second day in a row, you ask your friend to make a Cherry Pie. They scan their recipe book for "Cherry Pie", and take a look at the ingredients: PieCrust and Cherries. They look around in their kitchen, and to their horror, there is no pie crust to be found!


This would have been a disaster, but luckily, your friend happens to have a recipe for PieCrust in their recipe book! They use the PieCrust recipe, and then incorporate it into their CherryPie recipe. 

In makefile format, this looks like:
```Makefile
CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven at 375°

PieCrust: flour IceWater ... shortening
    Mix flour IceWater ... shortening
    Flatten into circular disk
```

So, your friend follows these steps:
1. Starts making CherryPie, notices we are missing PieCrust
2. Jumps to PieCrust, notices we have all the ingredients for PieCrust
3. Makes PieCrust using the recipe book instructions
4. Jumps back to CherryPie, notices we have all the ingredients for CherryPie
5. Makes CherryPie using the recipe book instructions. 

In a practical makefile example we might have something like this when compiling our app.
```Makefile
app: main.o file1.o file2.o
    g++ main.o file1.o file2.o -o app

main.o: main.cpp
    g++ main.cpp -c

file1.o: file1.cpp file1.h
    g++ file1.cpp file1.h -c

file2.o: file2.cpp file2.h
    g++ file2.cpp file2.h -c
```

Notice that all we have to do to compile the app, is still the following command:
```
$ make app
```

This is because it will scan for main.o, notice it doesn't exist, then *implicitly* call make main.o, scan for file1.o, then *implicitly* call make file1.o, and so on. Once the makefile has made all of these dependencies, it runs the final compilation command to compile "app"

### Making Changes to your Recipe!
After years (two days) of experience, your friend has mastered the art of baking Cherry Pies! They have learned that the pie usually comes out better if the oven is set to 425° instead. In fact, they have learned that **most of their recipes** come out better at a higher temperature. But that would mean they would have to change the temperature manually for every single recipe...

Unfortunately in the real world with physical copy recipe books, this is the sad truth of making changes to recipes on a large scale. However! We have technology! The Makefile recipe book allows us to define variables and use them throughout our recipes!

So, your friend changes the baking Makefile to look like:
```Makefile
OVEN_TEMP=425°

CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven at $(OVEN_TEMP)

PieCrust: flour IceWater ... shortening
    Mix flour IceWater ... shortening
    Flatten into circular disk to make PieCrust

ChocolateChipCookies: flour ... butter
    Mix Dry Ingredients
    Mix Wet Ingredients
    Pour Dry Ingredients into Wet Ingredients
    Bake in Oven at $(OVEN_TEMP)
```

In the case of compiling our app, we may make choices for our use of a compiler. g++ is one example of a compiler, as well as clang++. Sometimes we might want to compile with g++ or clang++ depending on each benefits, and our goal in compilation. 

So, we can define our makefile in the following way:
```Makefile
CXX=g++

app: main.o file1.o file2.o
    $(CXX) main.o file1.o file2.o -o app

main.o: main.cpp
    $(CXX) main.cpp -c

file1.o: file1.cpp file1.h
    $(CXX) file1.cpp file1.h -c

file2.o: file2.cpp file2.h
    $(CXX) file2.cpp file2.h -c
```

Now if we ever want to switch from g++ to clang++, we just change the value of CXX to clang++.

Often times we have compilation flags. One common one is ```-std=c++11``` which means "compile this program according to the c++11 standard". Another common flag is ```-Wall``` which means "show me **all** warnings that are generated when compiling". 

Instead of writing ```-std=c++11 -Wall``` everywhere, we can define that too, as a variable:

```Makefile
CXX=g++
CXX_FLAGS=-std=c++11 -Wall

app: main.o file1.o file2.o
    $(CXX) $(CXX_FLAGS) main.o file1.o file2.o -o app

main.o: main.cpp
    $(CXX) $(CXX_FLAGS) main.cpp -c

file1.o: file1.cpp file1.h
    $(CXX) $(CXX_FLAGS) file1.cpp file1.h -c

file2.o: file2.cpp file2.h
    $(CXX) $(CXX_FLAGS) file2.cpp file2.h -c
```


### Simplifying our Recipes
For our pie crust recipe, you'll notice we have an instruction that says, 
1. Mix flour IceWater ... shortening

However, we are really just mixing all of the ingredients in our ingredients list!
Your friend thinks writing this all out twice is unnecessary, and rewrites the recipe to say: 

```Makefile
PieCrust: flour IceWater ... shortening
    Mix Ingredients
    Flatten into circular disk to make RecipeName
```

Now, when your friend reads this recipe, they know that you need to mix all of the ingredients. You will also notice that we have changed "to make PieCrust" to "to make RecipeName". RecipeName refers to... the name of the recipe, to be found to the left of the colon. So, RecipeName becomes a shortcut to PieCrust

When compiling our app, you notice we also have this same problem of restating our dependencies (ingredients). So, we can simplify our make commands in the following way:

```Makefile
CXX=g++

app: main.o file1.o file2.o
    $(CXX) $^ -o $@

main.o: main.cpp
    $(CXX) $^ -c

file1.o: file1.cpp file1.h
    $(CXX) $^ -c

file2.o: file2.cpp file2.h
    $(CXX) $^ -c
```

Notice that ```$^``` is our way of saying "Ingredients" and ```$@``` is our way of saying "RecipeName"

### Keeping it Fresh!
Your friend is quite meticulous when it comes to baking. Every time your friend bakes something from their recipe book, they keep track of the freshness of every ingredient they used. So, every ingredient that appears in their recipe book has an associated freshness level, indicated via a timestamp. So, if our friend makes CherryPie at 11:00 a.m., PieCrust, along with all of its ingredients, has its freshness level updated to: 11:00am. Cherries are also updated in the same way. 

Now, lets say you ask your friend, for the third day in a row, to make you a Cherry Pie. Your friend looks at the Cherry Pie they made yesterday and says, 
> "Hmm... I already have this Cherry Pie that I made you yesterday, and all the ingredients in my pantry are of the same freshness as when I made the pie. Therefore, this pie is the freshest it can be. There is no need to make a new Cherry Pie, you can have this one instead!"

So, your friend decides **NOT** make a new Cherry Pie and serves you the already existing one. 

However, the next day, your friend takes a trip to the store and buys a brand spanking new can of Cherries. 

When they get back, you ask them, for the fourth day in a row, for a Cherry Pie. Your friend consults their ingredient list and says, 
> "Hmmm... I already have this Cherry Pie that I made you yesterday. However, these Cherries that I just bought from the store are fresher than the Cherries in that pie! Therefore, I will make you a brand new pie with these new Cherries I bought, so that you may experience the most freshest Cherry Pie possible!"


So, your friend **DOES** decides to make a new Cherry Pie and serves it to you. 

Makefiles in the programming world work in the same exact way. In our app example, if you run: ```make app```, makefile will remember the timestamps of each file at the time the command was ran. Timestamps only change if *a change is made to the file*. So, rather than making every recipe from scratch every time, makefile only re-makes the recipes which are **outdated**, saving on compilation time in most cases. 

Lets be a little more specific by considering our running makefile:

```
CXX=g++

app: main.o file1.o file2.o
    $(CXX) $^ -o $@

main.o: main.cpp
    $(CXX) $^ -c

file1.o: file1.cpp file1.h
    $(CXX) $^ -c

file2.o: file2.cpp file2.h
    $(CXX) $^ -c
```

Suppose we start fresh and run ```make app```. The last time file1.cpp was changed was 11:00 a.m. and the last time file2.cpp was changed was 11:00 a.m. So, the makefile remembers these respective timestamps after the first ```make app``` call. Since it is the first time, the makefile implicitly makes every recipe: main.o, file1.o, and file2.o. 

Now, if we run ```make app``` again without making any changes, Makefile will say:

```make: `app' is up to date.```

Now, lets make a change to file1.cpp at 11:05 a.m. and run ```make app```. 
You will notice makefile only outputs:

```
g++ file1.cpp file1.h -c
g++ main.o file1.o file2.o -o app
```

The makefile is **NOT** running:
```
g++ file2.cpp file2.h -c
g++ main.cpp -c
```

This is because main.o and file2.o is already up to date, so there is no need to recompile them. As mentioned before, this saves on compilation time! This is a big deal because sometimes compilation can take a while, and you'd rather not recompile files that don't need to be recompiled. 

### Preparing a Three Course Meal
This recipe book is pretty weird becaues, as we've seen, some recipes refer to other recipes (as in CherryPie and PieCrust). 

This means, asking your friend to bake a single recipe, can cause any number of other recipes to also be made. 

Now suppose you are planning a three course meal. 
1. Appetizer: GarlicBread
2. Main: Spaghetti
3. Dessert: CherryPie

Instead of asking your friend to first make GarlicBread, then make Spaghetti, and then make CherryPie (for the fifth day in a row), you can simply ask your friend, "Could you please make a three course meal". Three course meal in this case, is acting as its own "recipe", whose ingredients are other recipes in the recipe book. So, our makefile could look like:
```makefile
OVEN_TEMP=425°

threeCourseMeal: GarlicBread Spaghetti CherryPie

GarlicBread: garlic bread butter
    mix butter and garlic, spread on bread
    bake bread in oven at $(OVEN_TEMP)

Spaghetti: SpaghettiSauce spaghettiNoodles
    Mix SpaghettiSauce with spaghettiNoodles

CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven at $(OVEN_TEMP)
```

Now, the command: ```make threeCourseMeal```, will implicitly make all three components of the meal, including GarlicBread, Spaghetti, and CherryPie. 

As you might guess, this is something we often take advantage of as Makefile users. 

Consider the case where we have an "app" program along with a "testapp" program. Instead of running ```make app``` and ```make testapp``` separately, by convention we usually use a recipe called "all" like so:
```Makefile
CXX=g++

all: app testapp

testapp: test.o
    $(CXX) $^ -o $@

app: main.o file1.o file2.o
    $(CXX) $^ -o $@

main.o: main.cpp
    $(CXX) $^ -c

test.o: test.cpp
    $(CXX) $^ -c

file1.o: file1.cpp file1.h
    $(CXX) $^ -c

file2.o: file2.cpp file2.h
    $(CXX) $^ -c
```

Notice how there are no instructions for the "all" recipe, since it is only used to call other recipes. 

### Cleaning The Kitchen
After all the CherryPies your friend has made you, their kitchen is a mess! Your friend is a chef though, and tries to make everything, even everyday actions, into recipes. So, they have a recipe called "cleanKitchen" which is a recipe for cleaning their kitchen!

```Makefile
OVEN_TEMP=425°

CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven at $(OVEN_TEMP)

PieCrust: flour IceWater ... shortening
    Mix flour IceWater ... shortening
    Flatten into circular disk to make PieCrust

ChocolateChipCookies: flour ... butter
    Mix Dry Ingredients
    Mix Wet Ingredients
    Pour Dry Ingredients into Wet Ingredients
    Bake in Oven at $(OVEN_TEMP)

#... other recipes, # are comment tags by the way!

cleanKitchen:
    Dispose of excess CherryPie, PieCrust, and ChocolateChipCookies
```

Notice how there are no ingredients for this action. 

So now you ask your friend to ```make cleanKitchen```... but **OH NO!** your friend's pantry contains a real ingredient called "cleanKitchen". Maybe it isn't such a good idea to have recipes for every day actions.

When you ask them to ```make cleanKitchen``` your friend gets confused because according to the freshness list, cleanKitchen (the ingredient) is up to date. So instead of cleaning the kitchen, your friend doesn't do anything. Indeed your friend is having an existential crisis. 

To avoid this catastrophic event, your friend inserts a note to themselves in the recipe book, to remove the cleanKitchen entry from the freshness list. 

```Makefile
OVEN_TEMP=425°

CherryPie: PieCrust Cherries
    Lay Crust on Pie Pan
    Pour Cherries into Pie Pan
    Bake in Oven at $(OVEN_TEMP)

PieCrust: flour IceWater ... shortening
    Mix flour IceWater ... shortening
    Flatten into circular disk to make PieCrust

ChocolateChipCookies: flour ... butter
    Mix Dry Ingredients
    Mix Wet Ingredients
    Pour Dry Ingredients into Wet Ingredients
    Bake in Oven at $(OVEN_TEMP)

#... other recipes, # are comment tags by the way!
cleanKitchen is a PHONY!!!
cleanKitchen:
    Dispose of excess CherryPie, PieCrust, and ChocolateChipCookies
```

Now when your friend consults the recipe book, they know that cleanKitchen should actually refer to an everyday action instead of a real-life ingredient. Knowing that this is an everyday action means you don't need to consult the freshness entry. 

In a makefile, we also may have a ```clean``` recipe which removes files that would otherwise clutter our workspace. 

```Makefile
CXX=g++
targets=all testapp

all: $(targets)

testapp: test.o
    $(CXX) $^ -o $@

app: main.o file1.o file2.o
    $(CXX) $^ -o $@

main.o: main.cpp
    $(CXX) $^ -c

test.o: test.cpp
    $(CXX) $^ -c

file1.o: file1.cpp file1.h
    $(CXX) $^ -c

file2.o: file2.cpp file2.h
    $(CXX) $^ -c

.PHONY: clean all
clean:
    rm -f *.o $(targets)
```

Note that all is included in the .PHONY "recipe" since it too refers to an "abstract everyday event". "all", like "clean", is not associated with any particular result, and is only used to call other recipes. 

```rm -f *.o $(targets)``` is used to remove all files ending with a ".o", with the "*" character acting as a wildcard to match any pattern. $(targets) is invoked to remove all our executables. Finally, ```-f``` is a flag for ```rm```, which stands for "force". Meaning, we want to remove these files without any further confirmation. 

### A Note on other uses of a Makefile
To get the idea of a makefile across, here is another, non-compilation, based application of Makefiles. Remember, Makefiles act as a place for us to create aliases to commands. 

###### Note* I am not recommending you do this, I am just showcasing the functionality of a Makefile
Have you ever been so tired of the same three git commands over and over again?
```
git add .
git commit -m ":3"
git push origin main
```

Yea me too... I wish there was a way to invoke all these commands at once...

But then you realize, I can make a Makefile recipe for this! So you run to your makefile and type this in:
```
gitpush: 
    git add .
    git commit -m ":3"
    git push origin main
```

Notice that gitpush does not have any dependencies. All you have to do it type in the command: 
```
$ make gitpush
```

and your wishes will be executed. 

###### Note* Again, I don't really condone this, most of the time, you want to add particular files (not the entire directory), have good commit messages (not just ":3") and / or to push to different branches (not just main).

The gist is: Makefiles are a powerful tool to help us compile our c++ programs. Happy Makefiling!