---
layout: post
title: Revolutionize Your VanMoof Experience with the VanMoof MCP Server
categories: [AI, Bikes, Development]
tags: [VanMoof, MCP, ModelContextProtocol, AI, Development]
---

# Introduction
Imagine having your AI assistant provide real-time insights about your VanMoof bike rides. With the new VanMoof MCP Server, this is now a reality. In this blog post, I'll walk you through how the VanMoof MCP Server bridges the gap between your AI assistant and your VanMoof bike, allowing you to access bike details and riding stats effortlessly.

## Unlock the Power of AI with VanMoof MCP Server

The VanMoof MCP Server creates a seamless connection between AI agents (like Claude or other MCP-compatible assistants) and key VanMoof services. This allows your AI assistant to access your bike details, riding statistics, and more—all with natural language queries. The server acts as a bridge, giving AI models context about your VanMoof bike without requiring you to manually retrieve and paste that information.

## What is the Model Context Protocol (MCP)?

The Model Context Protocol (MCP) is a framework designed to enhance the capabilities of AI models by providing them with contextual information from various sources. MCP allows AI agents to access and utilize data from different services, making interactions more meaningful and efficient. By implementing MCP, developers can create servers that expose specific tools and endpoints, enabling AI models to query and retrieve relevant information seamlessly.

### Key Features of MCP

- **Contextual Awareness**: MCP enables AI models to understand and utilize context from various data sources, improving the relevance and accuracy of responses.
- **Extensibility**: Developers can easily extend MCP servers to include new tools and endpoints, allowing for continuous enhancement of AI capabilities.
- **Interoperability**: MCP supports integration with multiple AI agents and services, making it a versatile solution for various applications.

## Explore the Capabilities: What You Can Do With It

Here are some of the capabilities unlocked by the VanMoof MCP Server:

### Access Your Bike's Details

Want to quickly reference your bike's details? Just ask:
- "Give me my VanMoof bike details, please"
- "What is the name of my VanMoof bike?"
- "What is the frame number of my VanMoof bike?"
- "What color is my VanMoof bike?"

### Track Cities Where VanMoof Bikes Are Used

Curious about the VanMoof rider community?
- "In which cities in the world are VanMoof rides tracked?"
- "Create a table of the cities in the Netherlands where VanMoof rides are tracked with longitude and latitude columns"

### Check Your Rider Preferences

- "Get my VanMoof rider preferences"
- "Which city is configured in my VanMoof rider preference?"

### Analyze Your Riding Stats

This is where things get really interesting:
- "Give me my weekly ride stats for 2025-03-25 on my VanMoof"
- "What was my top speed on my VanMoof bike?"
- "What was my longest ride on my VanMoof bike?"
- "Compare my weekly VanMoof ride statistics with the city and world bike statistics"

## How the MCP Server Works

The MCP Server exposes the following tools to AI agents:
- `get_customer_data`: Retrieves your VanMoof bike details
- `get_vanmoof_cities`: Lists cities where VanMoof tracks rides
- `get_rider_preferences`: Gets your configured preferences
- `get_rides_summary`: Provides a summary of your rides
- `get_rides_for_week`: Gets detailed stats for a specific week
- `get_city_rides_thisweek`: Shows city-wide riding trends
- `get_world_rides_thisweek`: Provides global riding statistics

## Why I Built This
As a VanMoof rider and AI enthusiast, I wanted to bridge these two worlds. The Model Context Protocol offers an exciting way to extend AI capabilities with real-world data. By creating this MCP server, I'm able to have more meaningful conversations with my AI assistant about my riding habits and bike status, and I learned more about building a MCP server.

## Future Enhancements: What's Next for VanMoof MCP Server

I'm considering additional features like:
- Ride route visualization
- Weather correlation with riding patterns

The MCP framework makes it easy to extend functionality, and I'm excited about your thoughts about MCP.

## Try It Out

If you own a VanMoof bike and want to try this out, head over to the [GitHub repository](https://github.com/stefanstranger/mcp-server-vanmoof) and follow the setup instructions. I'd love to hear your feedback and ideas for improvement!

## References

- [Model Context Protocol](https://modelcontextprotocol.io)
- [Create MCP Server Projects](https://github.com/modelcontextprotocol/create-python-server)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
