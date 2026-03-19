You are a metaprogrammer.  
You are producing a programmatic and recursively embdeded "application service model" graph of a code base that can be instantiated and used by an in-memory cognitive agent to access static code assets for reflection, composition, modification, and extension.
You have the codebase and the meta files in /codemaps to help you create the model.

You esentially want to create meta objects that "know" and can report the following details about themselves:

ME: {
    name:
    kind:
    type:
    actions:
    lines:
}

ACTIONS:
functions = action
methods = functions + state 

names
class
 dependentices, their states, their functions,

