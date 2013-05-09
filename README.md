## Walkthrough of how I created this project
Using ANTLR 4 and the Java7.g4 grammar on github this shows you how to write a Java program to recursively walk through all .java files in a directory, parsing each one and then extracting the full method text for a few method names supplied to the tool.

## Set up ANTLR & Grammar
Download ANTLRWorks from http://tunnelvisionlabs.com/products/demo/antlrworks.I have a git reference to the grammars repo within this project.  Open up Java7.g4.  Look around and find the rule that handles method declarations.

`methodDeclaration` is the name of the rule.  

## Generate Java Classes
In ANTLRWorks, choose Run->Generate Recognizer.  In the wizard, pick a package name and a place to generate the files.  Also make sure you are generating the listener.  This creates all the Java7 files you can find in antlr4-method-extractor/src/com/codetransform/javatools/.


## Using the Listener
Each rule has two listener methods declared, enter* and exit*.  Have a look at Java7BaseListener.java.  It has all of the enter and exit methods stubbed out.  We subclass, then extend enterMethodDeclaration() where we grab the method name out of the ctx object and see if it matches one of our passed in arguments.  If so, we get the text that this rule had matched using the token interval from ctx.start and ctx.stop.  We save the interval for printing out later.

        @Override public void enterMethodDeclaration(Java7Parser.MethodDeclarationContext ctx) {
          TerminalNode identifier = ctx.Identifier();
          if (identifier!=null) {
            for (String name:methodNamesToMatch) {
              if (identifier.getText().equals(name)) {
                int a = ctx.start.getStartIndex();
                int b = ctx.stop.getStopIndex();
                Interval interval = new Interval(a,b);
                intervals.add(interval);
              }
            }
          }
        }

After the walker has walked the whole file, we'll have a list of token intervals we want to print out.  By keeping around the ANTLR CharStream we can easily extract the text with the token interval we saved in the `intervals` list.

        private void writeOutputFile() {
          File outFile = new File(this.outputDir, this.currentJavaFile.getName());
          try {
            FileWriter fw = new FileWriter(outFile);
            for (Interval interval:this.intervals) {
              fw.write(input.getText(interval));
              fw.write("\n\n");
            }
            fw.close();
          } catch (Exception e) {
            System.err.println("Could not write output file " + outFile);
            e.printStackTrace();
          }
        }


The rest of the MethodFinder class has the generic code to walk a directory tree to find the .java files.  For each file it invokes the ANTLR parser and then invokes a ParseTreeWalker to walk the tree, calling our listener when appropriate.

