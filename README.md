✈️ TripMate AI — A Multi-Agent-WorkFlow (Travel Planner)with LangGraph


TripPlanner AI is a web-based travel planner built with FastAPI and LangGraph. Describe a trip in natural language to get live flight-status information, hotel suggestions, and an AI-generated itinerary.

## Features

- Search live flight information with AviationStack. Flight status data may not include ticket prices.
- Find hotel suggestions with Tavily Search.
- Generate a practical, budget-aware itinerary and a formatted trip response with Groq.
- Save workflow conversation state in PostgreSQL using a LangGraph checkpointer.
- Use the browser interface to copy a plan or download it as a PDF.
- Access the JSON API and health-check endpoint directly.

## How it works

Each travel request runs through four LangGraph agents in sequence:

1. **Flight agent** looks up flight information for the requested route.
2. **Hotel agent** searches for hotel suggestions.
3. **Itinerary agent** combines the request and search results into a day-by-day plan.
4. **Final response agent** formats the complete travel response.

The browser UI is served by FastAPI and uses the `/api/travel` endpoint to submit requests.

## Requirements

- Python 3.11 or later
- PostgreSQL database
- API keys for [Groq](https://console.groq.com/), [Tavily](https://app.tavily.com/), and [AviationStack](https://aviationstack.com/)

`GROQ_API_KEY` and `DATABASE_URL` are required for the application to start. Tavily is used for hotel searches, and AviationStack is used for live flight information.

## Setup

1. Clone the repository and enter the project directory:

   ```bash
   git clone https://github.com/hanifsidhartha26/Multi-Agent-WorkFlow-with-LangGraph.git
   cd Multi-Agent-WorkFlow-with-LangGraph
   ```

2. Create and activate a virtual environment:

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```

   On Windows PowerShell, activate it with:

   ```powershell
   .venv\Scripts\Activate.ps1
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Create a `.env` file in the project root:

   ```dotenv
   GROQ_API_KEY=your_groq_api_key
   TAVILY_API_KEY=your_tavily_api_key
   AVIATIONSTACK_API_KEY=your_aviationstack_api_key
   DATABASE_URL=postgresql://username:password@host:5432/database
   DEFAULT_ORIGIN_IATA=DAC
   ```

   `DEFAULT_ORIGIN_IATA` is optional and defaults to `DAC` (Dhaka). The application adds `sslmode=require` to `DATABASE_URL` when it is not already specified.

   Keep real credentials out of source control. The repository's `.gitignore` excludes `.env`.

5. Start the development server:

   ```bash
   uvicorn app:app --reload
   ```

6. Open [http://127.0.0.1:8000](http://127.0.0.1:8000).

On startup, the application connects to PostgreSQL and initializes the LangGraph checkpoint tables.

## API

### `GET /health`

Returns a simple health status:

```json
{
  "status": "ok",
  "message": "AI Travel Planner API is running"
}
```

### `POST /api/travel`

Submit a travel request. `thread_id` is optional; reuse the ID returned by a previous response to continue using that checkpoint thread.

```bash
curl -X POST http://127.0.0.1:8000/api/travel \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Plan a 7-day trip to Japan from Bangladesh under 2000 USD",
    "thread_id": null
  }'
```

A successful response includes:

```json
{
  "success": true,
  "thread_id": "user_<generated-id>",
  "answer": "Formatted travel plan...",
  "flight_results": "Flight search results...",
  "hotel_results": "Hotel search results...",
  "itinerary": "Day-by-day itinerary...",
  "llm_calls": 4
}
```

An empty message returns HTTP `400`. Application or integration errors return HTTP `500` with an `error` message.

## Docker

The repository includes a Dockerfile that serves the application on port `8000`. Configure the required environment variables in the container at runtime.

```bash
docker build -t tripplanner-ai .
docker run --rm -p 8000:8000 \
  -e GROQ_API_KEY \
  -e TAVILY_API_KEY \
  -e AVIATIONSTACK_API_KEY \
  -e DATABASE_URL \
  -e DEFAULT_ORIGIN_IATA \
  tripplanner-ai
```

Ensure `.env` is excluded from the Docker build context before building; do not bake credentials into the image.

## Project structure

```text
.
├── app.py                 # FastAPI app, web page, and JSON endpoints
├── backend.py             # LangGraph workflow and PostgreSQL checkpointing
├── tools/
│   ├── flight_tool.py     # AviationStack flight lookup
│   └── tavily_tool.py     # Tavily hotel/search lookup
├── templates/
│   └── index.html         # Planner page
├── static/
│   ├── script.js          # Browser interactions and API client
│   └── style.css          # Page styles
├── requirements.txt
└── Dockerfile
```

## Notes

- AviationStack provides live flight/status information, not guaranteed ticket fares. Confirm schedules and prices with an airline or booking provider.
- Search results and itinerary suggestions depend on the availability and accuracy of the configured external services. Verify travel details before booking.
✈️ TripMate AI — A Multi-Agent Travel Planner with LangGraph
