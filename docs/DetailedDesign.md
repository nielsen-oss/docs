#Detailed design - Guidelines

So we did a high level design and have some diagrams of what we should do. But high level diagrams cannot be directly translated into working code. 
Here are our guidelines for approaching detailed design

##Understand Requirements
- Who will use this feature?
- What problem is it solving?
- How would we know it works correctly?

##Understand Responsibilities
Everything we write should ideally be written in one of the following type of components:
- Client - Requests and responses to our server
- Manager - flow management
- Engine - Business logic
- Resource Access - How do we access our data
- Utility - Something that could be used anywhere (even a coffee machine)

If you are not sure which it is - discuss (rule of thumb is that if you are not sure it should be in the engine)

*We have an interal utility repository for base implemetations of each of the above component. in the future we will add it here*

##Think in Patterns/Toolbox

After realizing where should we write our code we should think about what “tools” can we use.
Here are some examples:
- Repository pattern for resource access
- Validator convention in engines
- Function composition/chaining for managers
- Factory/Strategy for engines

Ideally, there is already a pattern/convention to solve what you are trying to somewhere in our codebases. If you think there isn't one, discuss.

##Follow Clean Code
- Read "Clean Code"
- Follow it




### Features

- Support Standard Markdown / CommonMark and GFM(GitHub Flavored Markdown);
- Full-featured: Real-time Preview, Image (cross-domain) upload, Preformatted text/Code blocks/Tables insert, Code fold, Search replace, Read only, Themes, Multi-languages, L18n, HTML entities, Code syntax highlighting...;
- Markdown Extras : Support ToC (Table of Contents), Emoji, Task lists, @Links...;
- Compatible with all major browsers (IE8+), compatible Zepto.js and iPad;
- Support identification, interpretation, fliter of the HTML tags;
- Support TeX (LaTeX expressions, Based on KaTeX), Flowchart and Sequence Diagram of Markdown extended syntax;
- Support AMD/CMD (Require.js & Sea.js) Module Loader, and Custom/define editor plugins;
