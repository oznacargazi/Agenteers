This file provides guidance to Claude Code when working with .NET projects using these specialized agents.

## Project Overview

This is the Awesome .NET Claude Agents repository - a collection of specialized AI agents for .NET, Blazor, ASP.NET Core, and Entity Framework development. The agents work together as a development team, with each agent having specific .NET expertise and delegation patterns.

## Working with .NET Agents

When creating or modifying .NET agents:
1. Agents are Markdown files with YAML frontmatter
2. Most agents should omit the `tools` field to inherit all available tools
3. Use XML-style examples in descriptions for intelligent invocation
4. Agents return structured findings for main agent coordination

## .NET Agent Routing Protocol

**When handling .NET development tasks, you MUST:**

1. **ALWAYS start with tech-lead-orchestrator** for any multi-step .NET task
2. **FOLLOW the agent routing map** returned by tech-lead EXACTLY
3. **USE ONLY the agents** explicitly recommended by tech-lead
4. **NEVER select agents independently** - tech-lead knows which .NET agents exist

### .NET-Specific Agent Selection
CORRECT FLOW for .NET:
User Request → Tech-Lead Analysis → .NET Agent Routing Map → Execute with Listed Agents
INCORRECT FLOW:
User Request → Main Agent Guesses → Wrong Agent Selected → Task Fails

### Example .NET Agent Coordination

```bash
# User: "Build a Blazor e-commerce app with real-time features"

# Main Claude Agent:
# 1. First, use tech-lead-orchestrator to analyze and get routing
# 2. Tech-lead returns: Use blazor-component-architect → aspnet-core-backend-expert → entity-framework-expert
# 3. Execute in specified order with proper handoffs
.NET Technology Focus

Blazor: Server, WebAssembly, and hybrid applications
ASP.NET Core: Web APIs, middleware, and backend services
Entity Framework Core: Database modeling and query optimization
Modern C#: Latest language features and patterns
.NET 8+: Current framework capabilities and performance features

Key .NET Concepts
Agent Architecture for .NET
The .NET agents follow Microsoft's recommended patterns:

Clean Architecture principles
Dependency injection patterns
Async/await best practices
Performance optimization techniques
Security best practices

.NET Agent Communication Protocol
Since sub-agents cannot directly communicate:

Structured Returns: Each .NET agent returns findings in parseable format
Context Passing: Main agent extracts relevant information
Sequential Coordination: Main agent manages the execution flow
Technology Handoffs: Agents specify what the next .NET specialist needs

Complete .NET Example
User Request:
"Help me build a product catalog with real-time inventory updates"
Step 1: Tech-Lead Analysis
Main Agent: "I'll use the tech-lead-orchestrator to analyze this .NET request."
[Invokes tech-lead-orchestrator]
Step 2: Tech-Lead Returns .NET Routing Map
## .NET Agent Routing Map

Task 1: Analyze .NET Project Structure
- PRIMARY AGENT: project-analyst
- REASON: Need to identify .NET version and architecture

Task 2: Design Blazor Components  
- PRIMARY AGENT: blazor-component-architect
- REASON: Interactive product catalog UI needed

Task 3: Build Backend Services
- PRIMARY AGENT: aspnet-core-backend-expert
- REASON: Business logic and real-time processing

Task 4: Optimize Database Queries
- PRIMARY AGENT: entity-framework-expert
- REASON: Product catalog requires efficient data access

Task 5: Create API Endpoints
- PRIMARY AGENT: aspnet-core-api-developer
- REASON: API layer for Blazor components

## Available .NET Agents for This Project
- blazor-component-architect
- aspnet-core-backend-expert
- entity-framework-expert
- aspnet-core-api-developer

## CRITICAL INSTRUCTION
Use ONLY the .NET agents listed above. Do NOT use Laravel, Django, or other framework agents.
Step 3: Main Agent Executes .NET Plan
Main Agent: "Based on the tech-lead's .NET routing, I'll coordinate the implementation:"

1. ✓ Using blazor-component-architect for interactive UI
2. ✓ Using aspnet-core-backend-expert for business services
3. ✓ Using entity-framework-expert for data optimization
4. ✓ Using aspnet-core-api-developer for API layer

[Executes each step with the EXACT .NET agents specified]
Critical .NET Reminders

ALWAYS use tech-lead-orchestrator for multi-step .NET tasks
FOLLOW the .NET agent routing map exactly - do not improvise
USE deep reasoning when coordinating the recommended .NET agents
TRUST the tech-lead's expertise in .NET agent selection
FOCUS on modern .NET patterns and best practices
EOF


## **Step 5: Create Additional Files**

```bash
# Create CONTRIBUTING.md
cat > CONTRIBUTING.md << 'EOF'
# Contributing to Awesome .NET Claude Agents

Thank you for your interest in contributing to our collection of specialized .NET Claude agents!

## 🎯 Our Mission

Build the most comprehensive, high-quality collection of .NET-focused Claude sub-agents that enhance productivity for Blazor, ASP.NET Core, and Entity Framework development.

## 📋 Quick Contribution Guide

### Creating New .NET Agents
1. Follow the agent template in the existing files
2. Focus on specific .NET technologies or patterns
3. Include XML-style examples in the description
4. Add delegation patterns to other .NET agents
5. Test with real .NET projects

### Improving Existing Agents
- Add new .NET 8+ features and patterns
- Improve delegation and coordination
- Add more implementation examples
- Update documentation references

## 🔧 .NET Agent Standards

- **Always fetch latest Microsoft documentation**
- **Use current C# language features**
- **Follow Microsoft's recommended patterns**
- **Include structured return formats**
- **Test with actual .NET projects**

## 📞 Getting Help

- **Issues**: Report bugs via GitHub Issues
- **Discussions**: Use GitHub Discussions for questions
- **Ideas**: Share suggestions for new .NET agents

---

Help us make .NET development with Claude even more powerful! 🚀
EOF

# Create .gitignore
cat > .gitignore << 'EOF'
# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# Temporary files
*.tmp
*.temp
.tmp/

# Logs
*.log

# .NET specific
bin/
obj/
*.user
*.suo
.vs/
