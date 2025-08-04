# Agenteers - .NET Claude Agents 🧑‍💻

> Specialized AI agents that bring expert .NET knowledge to Claude Code

Transform your .NET development workflow with four specialized agents designed specifically for Blazor, ASP.NET Core, and Entity Framework projects. Each agent provides deep expertise and seamlessly coordinates with others to deliver complete solutions.

**Inspired by and built upon the excellent work from [awesome-claude-agents](https://github.com/vijaythecoder/awesome-claude-agents) by @vijaythecoder**

## 🎯 What Makes Agenteers Different

Unlike general-purpose coding assistants, these agents are laser-focused on the .NET ecosystem:

- **Deep .NET Knowledge**: Each agent specializes in specific Microsoft technologies
- **Project Context Aware**: Automatically adapts to your existing .NET project structure  
- **Real Coordination**: Agents hand off work to each other intelligently
- **Production Ready**: Follows Microsoft's latest best practices and patterns

## 🛠 Your .NET Expert Team

| Agent | Specialty | When to Use |
|-------|-----------|-------------|
| **blazor-component-architect** | Interactive UI Components | Building Blazor Server/WASM apps, component libraries |
| **aspnet-core-backend-expert** | Business Logic & Services | Clean architecture, dependency injection, background jobs |
| **aspnet-core-api-developer** | Web APIs & Integration | REST APIs, authentication, OpenAPI documentation |
| **entity-framework-expert** | Database & Performance | EF Core optimization, complex queries, migrations |

## ⚡ Getting Started

**Step 1: Get the agents**
```bash
git clone https://github.com/oznacargazi/Agenteers.git
cd Agenteers
```

**Step 2: Link to Claude Code**
```bash
# Create symlink (updates automatically)
ln -sf "$(pwd)" ~/.claude/agenteers

# Or copy once (static)
cp -r . ~/.claude/agenteers
```

**Step 3: Try it out**
```bash
# Navigate to your .NET project
cd /path/to/your/dotnet/project

# Let the agents analyze and help
claude "use @agent-blazor-component-architect to improve my product list component"
```

## 💡 Real-World Examples

**Building a Feature End-to-End:**
```bash
claude "I need a real-time inventory system for my e-commerce site"
# → Agents coordinate: EF Expert → Backend Expert → API Developer → Blazor Architect
```

**Optimizing Performance:**
```bash
claude "My dashboard loads slowly, help optimize it"
# → EF Expert analyzes queries, Blazor Architect improves rendering
```

**Adding Authentication:**
```bash
claude "Add JWT auth to my existing API"
# → API Developer handles auth, Backend Expert updates services accordingly
```

## 🏗 Architecture Philosophy

These agents follow modern .NET principles:

- **Clean Architecture**: Separation of concerns, dependency inversion
- **Domain-Driven Design**: Rich domain models, bounded contexts  
- **Performance First**: Async/await, memory efficiency, query optimization
- **Security by Default**: Authentication, authorization, input validation
- **Testability**: Dependency injection, mocking, unit test patterns

## 📖 Documentation

- [Agent Usage Guide](docs/usage-guide.md) - How to get the most from each agent
- [.NET Patterns](docs/dotnet-patterns.md) - Best practices the agents follow
- [Troubleshooting](docs/troubleshooting.md) - Common issues and solutions

## 🤝 Contributing

Found a bug? Have an idea for a new .NET agent? Contributions welcome!

- **Issues**: [Report problems](https://github.com/oznacargazi/Agenteers/issues)
- **Discussions**: [Share ideas](https://github.com/oznacargazi/Agenteers/discussions)
- **Pull Requests**: Follow our [contribution guide](CONTRIBUTING.md)

## 🙏 Acknowledgments

This project builds upon the excellent foundation laid by:
- **[awesome-claude-agents](https://github.com/vijaythecoder/awesome-claude-agents)** - The original agent orchestration system
- **Microsoft .NET Team** - For the amazing .NET ecosystem
- **Claude Code Community** - For pushing the boundaries of AI-assisted development

## 📄 License

MIT License - Build amazing .NET apps with these agents!

---

**Ready to supercharge your .NET development?** ⭐ Star this repo and start building with your new AI team!

