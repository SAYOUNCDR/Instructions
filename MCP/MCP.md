- What is MCP Servers

  - The Why
  - The What
  - The How

- PS : News Letter on mail - Get daily

  - Own Newslettter
    - Solve the problem of making newsletter which can be sent by mail using MCP

- First how the Newsletter should look like must be clear

  - Introduction
  - Big story of the week
  - 3-5 Quick Updates
  - Top Research Papers
  - 1 Quick tutorial
  - Top Ai products of the week
  - Top X posts
  - Closing notes

- Clearly definig thow and what should look like the newsletter cause it will be donen by AI

  - Research Notes
  - Editing
  - Designing

- Tools

  - Claude Chatbot
  - Web search
  - Github
  - Google Drive
  - Gmail
  - Product Hunt

- The Why
  - Context Context is everything an Ai can see when it generates a response
  - There is no single source of context from where ai can get the context

Example :

- Let say a software engineer was assigned a task to impletement 2 factor authentication in a web app
  Steps:
  - He checks his jira tickets for what exactly is required
  - He checks his company documentation and codebase for any existing implementation
  - He checks slack channels for any discussions within the team
  - He can search on google for 2 factor authentication implementation
  - He can check github for some open source projects
- All these sources are context for the software engineer to complete his task
- Similarly an AI needs context from multiple sources to generate a good response

  - Currently there is no single platform which can provide context from multiple sources to an AI

- If Engineer have to implement 2FA via AI
  - He has to manually provide context from multiple sources to the AI
  - This is time consuming and inefficient because copy pasting from multiple sources is tedious just to create context for AI
  - In a way the Human just became API for the AI to get context from multiple sources just to complete a simple task

Problem :

- Time for Context Gathering >> Time for Task Completion

- Solution : MCP Servers

  - MCP Servers will provide a platform where multiple context sources can be connected to provide context to AI for task completion
  - Just like how API servers provide data to applications, MCP servers will provide context to AI for task completion
  - This will reduce the time for context gathering and increase the efficiency of AI in task completion

- MCP have two main components

  - MCP Client
  - MCP Server

- MCP Client

  - MCP Client will be the chatbot interface where user can interact with the AI models
  - User will provide the task to be completed via MCP Client

- MCP Server

  - MCP Server will be responsible for aggregating the context from multiple sources and providing it to the AI models for task completion.

- Why MCP is booming

  - Ai needs context from multiple sources to generate good responses and other tools need to make their MCP servers to provide context to AI

- MCP Architecture using first principles

  - Host -> Ai chatbot(could be owned or 3rd party like claude, chatgpt etc)

  - Server -> MCP Server (could be of slack, github, google drive, gmail, web search etc)

- Flow :
  Let say user asks the chatbot "Are there any new commits on my github repo ?"

  - User -> Host(Ai Chatbot) -> MCP Server (Github) -> Github API -> Commits Data -> MCP Server -> Host(Ai Chatbot) -> User

Here MCP server of github gave the context of commits data to the Ai chatbot to answer the user query

In reality Host (Ai chatbot ) never direclty talks to server host asks to client about query in high level then client converts it to mcp compatible query and sends it to MCP Server which then fetches data from the source and converts it to a language model compatible format and sends it back to Host via MCP client

Host can connect to multiple MCP servers but for host to connect to multiple MCP servers it needs to have multiple MCP clients

Client and MCP server has one to one mapping

Benefits of This Architecture

- Scalability : New MCP servers can be added easily without affecting the existing architecture
- Decoupling : Host and MCP servers are decoupled which allows independent development and deployment
- Flexibility : Host can connect to multiple MCP servers to get context from multiple sources

-Primitives [Things that server offer to host via client]

- Tools (Actions that host can perform via server)
- Resources (Structured Data that host can access via server)
- Prompts (Predefined queries that host can use via server)
