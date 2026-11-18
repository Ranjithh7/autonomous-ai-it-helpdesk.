# autonomous-ai-it-helpdesk.
IT teams spends 40% of time on repetitive L1 tickets. Industry needs an Autonomous AI agent that resolves password resets, software installs and access request without human involvement.
# Autonomous AI Agent for Intelligent IT Helpdesk Automation

## Project Overview
This repository contains the 7th-semester B.Tech mini-project development workspace for an Autonomous AI Helpdesk Agent. The system leverages Large Language Models (LLMs) and agentic loops to automate enterprise Level 1 (L1) support workflows securely.

## Core Modules Planned
- **Intent Processing Engine:** Classifies incoming natural language user requests.
- **Agentic Execution Loop:** Built on LangChain/CrewAI frameworks using the ReAct pattern.
- **Action & Automation APIs:** Python-based endpoints for simulating active workflows.

## Tech Stack
- **Language:** Python
- **Orchestration:** LangChain / CrewAI
- **Backend Framework:** FastAPI

## Architecture Diagram
- <img width="1536" height="1024" alt="Architecture" src="https://github.com/user-attachments/assets/352919d4-7afb-4e11-a499-6cef3109aee6" />
## Source Code 
- import sqlite3
from fastapi import FastAPI
from pydantic import BaseModel
from backend.agent import execute_it_agent
from fastapi.responses import HTMLResponse

app = FastAPI(title="IT Helpdesk Agent API")

class TicketRequest(BaseModel):
    user_email: str
    request_text: str

@app.post("/api/resolve-ticket")
def resolve_ticket(payload: TicketRequest):
    # 1. Execute the LangChain agent
    response = execute_it_agent(payload.user_email, payload.request_text)
    
    # 2. Save the resolved ticket to the database so it appears in the sidebar
    try:
        conn = sqlite3.connect("it_helpdesk.db")
        cursor = conn.cursor()
        
        # Ensure the table exists
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS tickets (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_email TEXT,
                request_text TEXT,
                status TEXT
            )
        """)
        
        # Insert the newly solved ticket
        cursor.execute(
            "INSERT INTO tickets (user_email, request_text, status) VALUES (?, ?, ?)",
            (payload.user_email, payload.request_text, "Resolved")
        )
        conn.commit()
        conn.close()
    except Exception as e:
        print(f"Failed to save to database: {e}")

    return {"status": "success", "response": response}

@app.get("/api/tickets")
def get_tickets():
    try:
        # Connect to your SQLite database
        conn = sqlite3.connect("it_helpdesk.db")
        cursor = conn.cursor()
        
        cursor.execute("SELECT * FROM tickets") 
        rows = cursor.fetchall()
        
        # Dynamically map the columns to a dictionary so Streamlit can read it
        column_names = [description[0] for description in cursor.description]
        tickets_list = [dict(zip(column_names, row))]
        
        conn.close()
        return tickets_list
    except sqlite3.OperationalError:
        # If the table doesn't exist yet, return an empty list so Streamlit doesn't crash
        return []
    except Exception:
        return []

# ==========================================
# Password Reset Web Page UI
# ==========================================
@app.get("/reset-password", response_class=HTMLResponse)
def password_reset_page():
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>Enterprise IT - Reset Password</title>
        <style>
            body { font-family: Arial, sans-serif; background-color: #f4f4f9; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
            .card { background: white; padding: 40px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); width: 300px; text-align: center; }
            input { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; }
            button { background: #007BFF; color: white; border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; width: 100%; font-weight: bold; }
            button:hover { background: #0056b3; }
            .success-text { color: #28a745; }
        </style>
    </head>
    <body>
        <!-- Password Form -->
        <div class="card" id="form-card">
            <h2>Reset Password</h2>
            <p style="font-size: 14px; color: #666;">Enter your new credentials below.</p>
            <form onsubmit="showSuccess(event)">
                <input type="password" id="new_pass" placeholder="New Password" required>
                <input type="password" id="confirm_pass" placeholder="Confirm Password" required>
                <button type="submit">Change Password</button>
            </form>
        </div>

        <!-- Success Message (Hidden by default) -->
        <div class="card" id="success-card" style="display: none;">
            <h2 style="font-size: 40px; margin: 0;" class="success-text">✅</h2>
            <h2 class="success-text">Password Changed!</h2>
            <p style="color: #333;">Your new password has been successfully synced with Active Directory.</p>
            <p style="font-size: 12px; color: #666;">You may now close this window.</p>
        </div>

        <script>
            function showSuccess(event) {
                event.preventDefault(); // Stops the page from crashing/refreshing
                
                var pass1 = document.getElementById('new_pass').value;
                var pass2 = document.getElementById('confirm_pass').value;
                
                if (pass1 !== pass2) {
                    alert("Passwords do not match. Please try again.");
                    return;
                }

                // Hide form and show success
                document.getElementById('form-card').style.display = 'none';
                document.getElementById('success-card').style.display = 'block';
            }
        </script>
    </body>
    </html>
    """
- The Source code used for the project Autonomous AI Agent for IT Help Desk
## Snaps of the UI, Problem Request & Solving
- <img width="1600" height="791" alt="image" src="https://github.com/user-attachments/assets/f2b38ffe-e3c4-4d10-90fc-8891a615b5c5" />
-<img width="484" height="429" alt="image" src="https://github.com/user-attachments/assets/ae9eba87-1eab-4ac4-a3c3-4cf86e15f3ad" />
-<img width="497" height="387" alt="image" src="https://github.com/user-attachments/assets/8aad160a-c22e-46d0-bab9-6475f1523d36" />


