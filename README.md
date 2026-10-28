# Cafe Order Management - AI Order Agent

A FastAPI-based backend service that provides an intelligent order-taking agent for a cafe. The agent uses AWS Bedrock (Nova) via LangChain, integrates with MCP (Model Context Protocol) tools/resources/prompts, maintains conversation state with Postgres checkpointer, and includes human-in-the-loop approval for placing orders.

## Architecture Overview

The application follows a layered architecture:

`
client/Code/src/
|-- main.py                 # FastAPI app, middleware, exception handlers
|-- settings.py             # Environment configuration (dataclass)
|-- agents/                 # AI agent implementation
|-- routes/                 # API route definitions
|-- services/               # Business logic layer
|-- repositories/           # Data access layer (SQLAlchemy)
|   \-- schemas/            # Database models
|-- models/                 # DTOs and domain models
\-- utils/                  # Config, exceptions, error logging
`

## Key Features

- **AI Order Agent**: LangChain agent powered by AWS Bedrock (Nova model) with system prompts loaded from MCP server
- **MCP Integration**: Connects to MCP server via streamable HTTP to load tools, resources, and prompts
- **Conversation Memory**: Uses LangGraph's Postgres checkpointer for persistent conversation state per customer
- **Context Summarization**: Automatic summarization middleware that preserves user name, order items, and favorite dish
- **Human-in-the-Loop**: Interrupts for approval when placing orders
- **REST API**: FastAPI endpoints for agent interactions
- **Error Handling**: Centralized exception handling with custom error codes and standardized API responses
- **PostgreSQL**: Database for storing orders, items, order items, and error logs

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /training/api/v1/order_agent | Process user query through the order agent |

**Request Body:**
`json
{
  "customer_id": 123,
  "user_query": "I'd like to order a coffee"
}
`

## Database Schema

- **items**: Menu items with name, price, active status
- **orders**: Customer orders with total price and status (ORDERED, PREPARING, DELIVERED)
- **order_items**: Junction table linking orders and items with quantities
- **errorlogs**: Error tracking with filename, function name, and error message

## Environment Configuration

Create a .env file in client/Code/src/ with the following variables:

`env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cafe_chatbot_agent
DB_USERNAME=postgres
DB_PASSWORD=your_password

# Application
HOST=0.0.0.0
PORT=8080
LOG_LEVEL=INFO

# AWS Bedrock
REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
MODEL_ID=your_bedrock_model_arn
PROVIDER=amazon

# MCP Server
MCP_URL=http://127.0.0.1:8000/mcp
`

## Running the Application

1. Navigate to the source directory:
`ash
cd client/Code/src
`

2. Install dependencies

3. Set up PostgreSQL database and ensure MCP server is running

4. Run the application:
`ash
python main.py
`

Or with uvicorn directly:
`ash
uvicorn main:main --host 0.0.0.0 --port 8080 --reload
`

## Notes

- The agent uses thread-based conversation history via customer_id as thread_id
- Summarization triggers after 300 tokens, keeping last 4 messages
- Human approval is required for place_order tool calls
- Database tables are auto-created by LangGraph's PostgresSaver on startup
