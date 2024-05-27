# How to make a good AI planner Agent. Part 2
Making hierarchical plans with Autogen

In the last Article, we demonstrated how to construct an effective planning Agent. 
That agent sits within an Autogen groupchat that can translate their plans into structured function calls. This creates a module that can: 
* Create a plan
* Evaluate a plan
* Select actions
* Trigger execution for those Actions. 




![Agent speaking transition - Copy of goal setting (9)](https://github.com/Aretai-Leah/Task_decomposition/assets/147453745/b93f1a4c-d42f-4122-a6d5-d9a9e3519fae)



















In this article we will show how we can stack this structure indefinitely to break very complex requests into discrete parts that can be actioned by individual agents or via traditional code. 






![Agent speaking transition - Copy of goal setting (14)](https://github.com/Aretai-Leah/Task_decomposition/assets/147453745/01679b7d-ac46-4427-9d96-528ca0e283eb)





## Connecting the dots
Each of the Modules ends with a function call. We connect the Modules via these function calls. The output of the function is passed as the initial message to the lower level module.







![Agent speaking transition - Copy of goal setting (16)](https://github.com/Aretai-Leah/Task_decomposition/assets/147453745/4267db7c-ee3a-4a2a-81f7-39c3ddb19655)













By doing this, the Task Decomposition Agent in the lower module can then break down the task further and then perform the atomic request. Which, In this example  is a bing search for the birthdate of one of the Beatles. 

For the return path; firstly the results are collated by the agent performing the function calls. Next the response agent then returns the results and ends the groupchat in the lower module. This passes the response back to the upper module via the function return. 

''''	
***** Response from calling tool (call_nVp0kN0jn3SQ85OO5i57dowK) ***** 


- **John Lennon** was born on **October 9, 1940**.
- **Paul McCartney** was born on **June 18, 1942**.
- **George Harrison** was born on **February 25, 1943**.
- **Ringo Starr** was born on **July 7, 1940**.


TERMINUS_W
''''


Note the very small number of tokens in the above response. The entire workflow to generate that answer has been obfuscated behind the function call. All the secondary task decomposition, agent coordination and API calling is not visible to the upper module’s group chat. All they see is the actionable results they need. 

## All you need to do is maintain Attention

“Attention is all you need” is a truism at this point. However, once you have it, you need to maintain it. Complex tasks often fail as the context window gets crowded with information. 
It becomes very difficult for the agent to both discern what the actual task is and to find key information. Both of these problems are addressed through the use of hierarchical planning and execution modules. 

The use of function calls ensures that only actionable information is introduced back into the context window.  With only the required details being returned, there are fewer tokens for subsequent agents to get distracted by. We focus the attention of the system with a clear and executable plan and maintain it by only feeding in new data required to continue with the plan. This is performed at 

Lets look at a different example. An essay writing AI. 

This system has 5 functions. 
Create an essay file
Write an Introduction (Text generation)
Write a Paragraph (Text generation)
Write a Conclusion (Text generation)
Review the essay


![Agent speaking transition - Writing](https://github.com/Aretai-Leah/Task_decomposition/assets/147453745/379556ba-b8ff-4860-9cd9-55377aba2632)



Naturally, writing an essay is a token intensive activity. Each text section may be written, edited and rewritten multiple times (just like this article!)  . However, by separating out the text generation modules from the planning module we can ensure that only usable information is returned.

This is an example of the response from the introduction writing module. 

''''
{
"status": "success",
"message": "The introduction has been successfully updated in the essay document."
"TERMINUS_INTRO": "TERMINUS_INTRO"
}'
''''


It took about 10 inference calls to create the Introduction text. Additionally, the inference call that generated the final version of the text took 5656 tokens. That’s a lot of noise we have removed from the planning module’s context window. 

The end result might be a 1500 word essay, but the upper Module Agents only know that each of the sub tasks have been completed successfully. 

## Hierarchical Nested planning

This modular approach allows for a practical degree of task decomposition to occur at each level without the planner needing to understand the minutiae of the action space for each capability. 

As per the writing example, The upper module decomp agent knows that there are file creating, text generating and text reviewing functions. It doesn’t need to know more than that. It only needs to provide sufficient context for the next tier down to complete its action. This iterates downwards until we arrive at an atomic task that can be completed in a single inference call. 

This method keeps agents on track with tasks they can actually complete. This is how you can solve complex tasks with generative AI. 
