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
2. You scan the recipe's dependency list and check if you have every single one
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

Notice that ```$^``` is our way of saying "Ingredients" and ```$@``` is our way of saying "RecipeName"


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