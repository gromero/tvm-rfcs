- Feature Name: project_api_extend_metadata
- Start Date: 2021-09-09
- RFC PR: [apache/tvm-rfcs#0000](https://github.com/apache/tvm-rfcs/pull/0000)
- GitHub Issue: [apache/tvm#0000](https://github.com/apache/tvm/issues/0000)

[RFC][Project API] Extend metadata in ProjectOption

# Summary
[summary]: #summary

One paragraph explanation of the feature.

----

Extend metadata associated with options of methods provided by the Project API.


# Motivation
[motivation]: #motivation

Why are we doing this? What use cases does it support? What is the expected outcome?

----

Currently the option metadata provided by the Project API are insufficent to
allow building easily and automatically command line parses used by CLI tools,
like TVMC.

The metadata available for the project options, stored in instances of the 
ProjectOption class, A) don't provide a list of the API methods which support
the options, B) don't allow to determine if the options are required or
optional, C) and don't provide a default value if one is used by the Project
API server. As a consequence it complicates the integration with command line
interfaces that need to create command line arguments based on the availables
options on a given platform.

This RFC proposes to extend the existing metadata with four new members in
ProjecOption ('required', 'optional', 'type', and 'default') to address the
issues A, B, and C, easing the integration of Project API with CLI tools like
TVMC.


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the language and you were teaching it to a TVM user. 

That generally means:

- Introducing new named concepts.
- Explaining what the feature enables (hint: think in terms of examples).
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.

For internal RFCs (e.g. for compiler internals), this section should focus on how core contributors s
hould think about the change, and give examples of its concrete impact. 

For policy RFCs, this section should provide an example-driven introduction to the policy, 
  and explain its impact in concrete terms.

----

Below it is explained in detail the need and properties of the four new members
('required', 'optional', 'type', and 'default') proposed to be added to the
ProjectOption class to extend the project option metadata available via the
Project API 'server_info_query'.

Modals like "must", "may" and similar ones are interpreted in this RFC          
accordingly to the semantics defined by the IETF RFC-2119, 1997.
                                                                                
On "required" and "optional" metadata                                           
                                                                                
Currently even though all options available for a given project can be          
discovered via the Project API 'server_info_query' there is no way to known     
which option belongs (or applies) to which API method (like generate_project,      
build, flash, and open_transport methods).                                              
                                                                                
This is fine when the user knowns beforehand which method accepts a set         
of options, so it's possible to manually select which options will be           
passed to a given API method, like when using the API in a Python script.       
                                                                                
However that's a problem when the API user (e.g. TVMC) needs to automatically   
determine the options available for the API methods, like when automatically    
building a command line parser with subcommand domains that closely mapped to   
the API methods (e.g. subcommands to create, build, and flash a project).       
                                                                                
Moreover, currently it's impossible to determine which option is required       
and which one is optional, so it would be at least necessary for the API user to
build a static table with all options available on a given project stating      
which option is required and which one is optional for the project. This is
impractical to maintain and  would result in the API user having to update the
static table every time an option is added, removed, or modified in the Project
API server.                
                                                                                
Hence to ease the automatic detection of the options available on each Project  
API method the following two new metadata are proposed: 'required' and
'optional'.  
                                                                                
Both will contain a list of method names for which the option is available,
either as a required option (if in 'required' list), or as an optional option
(if in 'optional' list). At least one API method must be listed in 'required' or
in the 'optional list. A method name must be listed only in the 'required' or in
the 'optional' list, i.e. an option can't be require and optional at the same
time for given API method. An option can be required for a method and optional
for another method.                                                    
                                                                                
The elements in the lists 'required' and 'optional' must be in the set of method
names implemented by the ProjectAPIClient class and that have the parameter     
'options' defined. These methods are dispatched to the server, which implements 
the server counterparts to properly handle the client dispatches and            
ultimately defines the options available for each API method. The current method
name that satisfy these criteria are 'generate_project', 'build', 'flash', and  
'open_transport'.                                                               
                                                                                
'required' metadatum or 'optional' metadatum (or both) must be provided for
every option.                                                                   
                                                                                
On "type" metadatum                                                             
                                                                                
The option type can sometimes be determined implicitly by what's return         
in metadatum 'choices', but this not ideal. For example, for option 'verbose' it
would be possible to infer it is a boolean option and therefore it can be       
converted to a command line flag if metadatum 'choices' is a couple             
of True and False. Nonetheless that would lead to cumbersome logic at API user  
side (e.g. TVMC) to infer the option type, like iterating over the tuple        
elements to search for True or False. This can be solved directly if the option 
type is returned explicitly with the option.                                      
                                                                                
Thus adding a 'type' metadatum allows a much simpler way for the API users to   
determine the type of an option when that is necessary for various reasons, like
when building a command line parser based on the available project options.     
                                                                                
The following types, passed as strings, are proposed for the 'type' metadatum:  
"bool", "str", "int", and "float".                                              
                                                                                
'type' metadatum is required, hence must be provided for every option.          
                                                                                
On "default" metadatum                                                          
                                                                                
Sometimes an option has a default value but currently there is no way to        
determine it.                                                                   
                                                                                
For some interfaces (e.g. TVMC)        


# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, 
and explain more fully how the detailed proposal makes those examples work.

---- 

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
