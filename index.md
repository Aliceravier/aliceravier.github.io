# Scala-Mungo tutorial

## Introduction
Scala-Mungo is a tool which lets you add a protocol/typestate definition to your classes. 
In this tutorial you will learn how to install Scala-Mungo, create a protocol for a class see an example of a program using that class.

## Installing Scala-Mungo

# SBT
Copy in these lines into your build.sbt file:

`
resolvers += Resolver.bintrayRepo("aliceravier", "maven")

autoCompilerPlugins := true

addCompilerPlugin("org.me" % "scala-mungo-prototype_2.13" % "1.4")
val root = ABSOLUTE-PATH-TO-YOUR-PROJECT
scalacOptions += "-P:GetFileFromAnnotation:"+root

libraryDependencies += "org.me" % "scala-mungo-prototype_2.13" % "1.4"
`
Instead of ABSOLUTE-PATH-TO-YOUR-PROJECT, put in the absolute path to your project, i.e. the path to the build.sbt file.
It should look like this:
`val root = val root = "C:\\Year five\\Scala-Mungo-dir\\Test"`
This is used for the plugin to find the location of the protocols you will define.

## Creating a protocol
A protocol consists of states that instances of the class can be in, and transitions between the states facilitated by method calls. 
All protocols must start in the "init" state and end in the "end" state.

# Specification

- The protocol must be defined inside an object which extends "ProtocolLang" and the code for the protocol must be in the main function of that object.

- States are defined with the "in" function which takes a String: the name of the state. Example: `in("init")`

- Transitions from a state can be added underneath a state definition with the "when" method which takes a String of the method signature as an argument, and the "goto" method which takes a String of the state to transition to as an argument. Example: `when("giveMoney(Int)") goto "moneyGiven"` (note that parenthesis aren't needed for the goto method's argument).

- To write a transition which depends on the return value of a method, the "at" and "or" methods are used. The "at" method specifies the return value which enables the transition and takes the return value as a String as an argument. The "or" method lets you add a different return value and takes the String of the next state to go to for the different return value. You can keep adding "or"s to add as many return values as you want. Example:
`
when("authorise()") goto 
    "Authorised" at "true" or 
    "Unauthorised" at "false"
`

- Protocols must contain a unique "init" state which will be the state given to an instance when it is initialised. \newline

- Protocols must contain a unique "end" state which indicates protocol completion, that is, at the end of a program, the object should be in the "end" state. All states must have a path of transitions between themselves and the "end" state.

- You must end your protocol with the "end()" method.

- The protocol file must be called by the name of the object which the protocol is defined it. Example: a protocol defined in the object "ATMProtocol" should have "ATMProtocol.scala" as the file name.


# Example
Let's say we want to create an ATM object which should follow a certain protocol. We want it to be able to take a card in, check if it's authorised, and then give money if it is or eject the card if not. Then we want the transaction to be available again. We need to include an "end" state which we can place right next to the "init" state, where all the transactions should start from.
We come up with the following state machine:
![ATM state machine](src)

Now we can write our protocol. We need to extend "ProtocolLang" and create a main method:
```markdown
import ProtocolDSL.ProtocolLang

object ATMProtocol extends ProtocolLang with App{

}
```
This needs to be in a file called "ATMProtocol.scala".

Then we can define the states we want:
```markdown
import ProtocolDSL.ProtocolLang

object ATMProtocol extends ProtocolLang with App{
  in("init")
  in("CardIn")
  in("Authorised")
  in("ShouldEject")
  in("end")
}
```

Then we can define the transitions for all the states:
```markdown
import ProtocolDSL.ProtocolLang

object ATMProtocol extends ProtocolLang with App{
  in("init")
  when("takeCard()") goto "CardIn"
  in("CardIn")
  when("authorise()") goto 
    "Authorised" at "true" or 
    "ShouldEject" at "false"
  in("Authorised")
  when("giveMoney()") goto "ShouldEject"
  in("ShouldEject")
  when("eject()") goto "end"
  in("end")
  when("beginNewTransaction()") goto "init"
 }
```

And finally we add the "end()" method at the end of the protocol:
```markdown
import ProtocolDSL.ProtocolLang

object ATMProtocol extends ProtocolLang with App{
  in("init")
  when("takeCard()") goto "CardIn"
  in("CardIn")
  when("authorise()") goto
    "Authorised" at "true" or
    "ShouldEject" at "false"
  in("Authorised")
  when("giveMoney()") goto "ShouldEject"
  in("ShouldEject")
  when("eject()") goto "end"
  in("end")
  when("beginNewTransaction()") goto "init"
  end()
}
```

Then we can run the protocol, which should create a file called "ATMProtocol.ser" inside a "compiledProtocols" directory in the project root. 
Now the plugin can find it and use it to check our ATMs are running correctly!

## Using a protocol in a program
Adding a protocol to a class in the program is easy. You only need to add the @Typestate annotation to the class which should be following the protocol. 
The annotation takes a String of the name of the object which the protocol was defined in as an argument.
In our ATM example:

```markdown
import compilerPlugin.Typestate

@Typestate("ATMProtocol")
class ATM(){...}
```

Here is an example program which uses the ATMProtocol defined above and represents the protocol:

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Aliceravier/aliceravier.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
