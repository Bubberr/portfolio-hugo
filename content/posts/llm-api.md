---
title: LLM API – AI Assignment Assessment
categories: ["AI", "Java"]
tags: ["llm", "ai", "java", "react", "groq", "api"]
date: 2026-04-29
draft: false
showauthor: true
authors:
  - Bubberr
---

## Building an AI-Powered Assignment Assessment Tool

A full-stack web application that uses a large language model to give students structured, rubric-based feedback on their internship reports (*praktikrapporter*) for the Datamatiker education programme in Denmark.

---

The tool lets a teacher or student paste an assignment text into a web form. The backend sends it — together with a predefined rubric — to a Groq-hosted LLM, which returns structured JSON feedback. The frontend then renders this as a clear, colour-coded report.

The assessment covers five weighted criteria drawn from the Datamatiker study programme:

| Criterion | Weight |
|---|---|
| Opfyldelse af læringsmål fra studieordningen | 25 % |
| Faglig refleksion og teorianvendelse | 25 % |
| Personlig og professionel udvikling | 20 % |
| Praktikkens udbytte for virksomhed og studerende | 20 % |
| Struktur, sprog og formalia | 10 % |

Each criterion is scored **Lav / Middel / Høj** with written justification referencing specific parts of the text.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend language | Java 17 |
| Web framework | Javalin 6.3 |
| HTTP client | OkHttp 4.12 |
| JSON | Jackson Databind 2.17 |
| Build tool | Maven |
| Frontend | React 18 + Vite 5 |
| LLM provider | Groq API (OpenAI-compatible) |
| Default model | `llama-3.1-8b-instant` |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    React Frontend                        │
│  AssessmentForm  ──►  App  ──►  AssessmentResult        │
└────────────────────────┬────────────────────────────────┘
                         │  POST /api/assess  (JSON)
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  Javalin REST API  :7070                 │
│                                                         │
│  AppConfig (CORS + routes)                              │
│       │                                                 │
│       ▼                                                 │
│  AssessmentController                                   │
│       │  validates request                              │
│       ▼                                                 │
│  AssessmentService                                      │
│       │  loads rubric.json, builds prompt               │
│       ▼                                                 │
│  LLMService  ──►  Groq API  ──►  llama-3.1-8b-instant  │
└─────────────────────────────────────────────────────────┘
```

### Data flow

1. User submits assignment text via the React form.
2. `AssessmentController` validates the request body is non-empty.
3. `AssessmentService` reads the rubric from `rubric.json` and builds a structured prompt that includes the full rubric and the assignment text.
4. `LLMService` calls the Groq API with a system prompt that instructs the model to return **only** a valid JSON object.
5. The JSON response is stripped of any markdown code fences and deserialised into `AssessmentResponse`.
6. A disclaimer is appended and the response is returned to the frontend.
7. `AssessmentResult` renders the result as cards: overall level badge, per-criterion cards, strengths/weaknesses columns, improvement suggestions, and dialogue questions.

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/assess` | Submit assignment text, receive structured assessment |
| `GET` | `/api/rubric` | Return the full rubric as JSON |
| `GET` | `/api/health` | Health check — returns `OK` |

### POST `/api/assess`

**Request body:**
```json
{
  "assignmentText": "The full text of the student's report..."
}
```

**Response body:**
```json
{
  "overallAssessment": "Short summary of the assessment...",
  "overallLevel": "Middel",
  "criteriaFeedback": [
    {
      "criterionName": "Opfyldelse af læringsmål fra studieordningen",
      "level": "Høj",
      "feedback": "The student clearly documents..."
    }
  ],
  "strengths": ["..."],
  "weaknesses": ["..."],
  "improvementSuggestions": ["..."],
  "dialogQuestions": ["..."],
  "disclaimer": "Dette er en vejledende AI-baseret vurdering og ikke en officiel eller endelig bedømmelse."
}
```

---

## Project Structure

```
llm-api/
├── src/main/java/dat/
│   ├── Main.java                        # Entry point – starts Javalin on port 7070
│   ├── config/
│   │   └── AppConfig.java               # CORS config + route registration
│   ├── controllers/
│   │   └── AssessmentController.java    # Request validation and response handling
│   ├── services/
│   │   ├── AssessmentService.java       # Rubric loading, prompt construction
│   │   └── LLMService.java              # Groq API integration via OkHttp
│   ├── dtos/
│   │   ├── AssessmentRequest.java
│   │   ├── AssessmentResponse.java
│   │   └── CriterionFeedback.java
│   └── models/
│       ├── Rubric.java                  # Rubric model + prompt serialisation
│       └── RubricCriterion.java
├── src/main/resources/
│   └── rubric.json                      # Rubric definition (weights + level descriptions)
├── frontend/
│   ├── src/
│   │   ├── App.jsx                      # State management + API calls
│   │   └── components/
│   │       ├── AssessmentForm.jsx       # Text input form with character count
│   │       └── AssessmentResult.jsx    # Result cards with level badges
│   ├── index.html
│   └── vite.config.js
├── .env.example                         # Required environment variables
└── pom.xml
```

---

## Getting Started

### Prerequisites

- Java 17+
- Maven 3.8+
- Node.js 18+
- A [Groq](https://console.groq.com) API key (free tier available)

### 1. Configure environment

```bash
cp .env.example .env
# Edit .env and add your Groq API key:
# GROQ_API_KEY=your_key_here
# LLM_MODEL=llama-3.1-8b-instant
```

### 2. Start the backend

```bash
mvn compile exec:java -Dexec.mainClass="dat.Main"
# Server runs on http://localhost:7070
```

### 3. Start the frontend

```bash
cd frontend
npm install
npm run dev
# Frontend runs on http://localhost:5173
```

---

## Design Decisions

**Groq over OpenAI** — Groq's inference API is OpenAI-compatible, free to get started with, and significantly faster for open-weight models like Llama 3.1. The `LLMService` can be pointed at any OpenAI-compatible endpoint by changing `API_URL`.

**Rubric as JSON, not code** — The rubric lives in `rubric.json` so it can be edited without recompiling. `Rubric.toPromptString()` serialises it into a human-readable block that the LLM understands reliably.

**Strict JSON output** — The system prompt instructs the model to return *only* a JSON object. `LLMService.cleanJsonResponse()` strips markdown code fences as a safety net, since some models wrap JSON in triple backticks despite instructions.

**No database** — The application is stateless. Each request is self-contained: rubric + assignment text → LLM → response. This keeps deployment simple and avoids storing student data.

---

## Deployment

To deploy, add the Maven Shade Plugin to `pom.xml` to produce a fat JAR:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.5.0</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals><goal>shade</goal></goals>
          <configuration>
            <transformers>
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>dat.Main</mainClass>
              </transformer>
            </transformers>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

Then:

```bash
mvn package                          # → target/llm-api-1.0-SNAPSHOT.jar
cd frontend && npm run build         # → frontend/dist/
```

The JAR can be run on any server with `java -jar llm-api-1.0-SNAPSHOT.jar`. The frontend `dist/` folder can be deployed to Netlify, Vercel, or served via nginx. For a single-service deployment, Javalin can be configured to serve the `dist/` folder as static files.
