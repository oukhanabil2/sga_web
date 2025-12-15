
5. **Commit new file**

## 🏗️ **ÉTAPE 3 : CRÉER LA STRUCTURE DES DOSSIERS**

**Sur GitHub, on ne peut pas créer des dossiers vides. On va créer un fichier dans chaque dossier pour les créer automatiquement.**

### **3.1 Créer `backend/main.py`**
1. **Add file → Create new file**
2. **Nom** : `backend/main.py`
3. **Contenu** (copiez tout) :

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import sqlite3
import os
from datetime import date

app = FastAPI(title="SGA API", version="1.0.0")

# Autoriser le frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Chemin de la base de données
DATABASE_PATH = "database/planning.db"

def get_db():
    """Connexion à la base de données"""
    if not os.path.exists(DATABASE_PATH):
        raise HTTPException(status_code=500, detail="Base de données non trouvée")
    
    conn = sqlite3.connect(DATABASE_PATH)
    conn.row_factory = sqlite3.Row
    return conn

@app.get("/")
def root():
    return {
        "application": "SGA Web API",
        "version": "1.0.0",
        "status": "running",
        "endpoints": {
            "agents": "/api/agents",
            "stats": "/api/stats",
            "planning": "/api/planning/{mois}/{annee}"
        }
    }

@app.get("/api/agents")
def get_agents():
    """Récupère tous les agents actifs"""
    conn = get_db()
    cursor = conn.cursor()
    
    try:
        cursor.execute("""
            SELECT code, nom, prenom, code_groupe, date_entree 
            FROM agents 
            WHERE date_sortie IS NULL 
            ORDER BY code_groupe, code
        """)
        agents = cursor.fetchall()
        return [dict(agent) for agent in agents]
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        conn.close()

@app.get("/api/stats")
def get_stats():
    """Statistiques pour le tableau de bord"""
    conn = get_db()
    cursor = conn.cursor()
    
    stats = {
        "total_agents": 0,
        "agents_par_groupe": {},
        "presents_aujourdhui": 0,
        "date": date.today().isoformat(),
        "status": "success"
    }
    
    try:
        # Agents par groupe
        cursor.execute("""
            SELECT code_groupe, COUNT(*) as count 
            FROM agents 
            WHERE date_sortie IS NULL 
            GROUP BY code_groupe
        """)
        
        for groupe, count in cursor.fetchall():
            stats["agents_par_groupe"][groupe] = count
            stats["total_agents"] += count
        
        # Présents aujourd'hui
        aujourdhui = date.today().isoformat()
        cursor.execute("""
            SELECT COUNT(DISTINCT code_agent) 
            FROM planning 
            WHERE date = ? AND shift IN ('1', '2', '3')
        """, (aujourdhui,))
        
        result = cursor.fetchone()
        stats["presents_aujourdhui"] = result[0] if result and result[0] else 0
        
    except Exception as e:
        stats["status"] = "error"
        stats["message"] = str(e)
    finally:
        conn.close()
    
    return stats

@app.get("/api/planning/{mois}/{annee}")
def get_planning_mois(mois: int, annee: int):
    """Planning pour un mois donné"""
    conn = get_db()
    cursor = conn.cursor()
    
    try:
        cursor.execute("""
            SELECT DISTINCT code_agent 
            FROM planning 
            WHERE strftime('%m', date) = ? 
            AND strftime('%Y', date) = ?
        """, (f"{mois:02d}", str(annee)))
        
        agents = [row[0] for row in cursor.fetchall()]
        
        return {
            "mois": mois,
            "annee": annee,
            "agents_count": len(agents),
            "agents": agents[:10] if agents else []
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        conn.close()

if __name__ == "__main__":
    import uvicorn
    print("🚀 SGA Web API démarrée sur http://localhost:8000")
    print("📚 Documentation: http://localhost:8000/docs")
    print("📊 Endpoints:")
    print("   - GET /api/agents → Liste des agents")
    print("   - GET /api/stats → Statistiques")
    print("   - GET /api/planning/{mois}/{annee} → Planning")
    
    uvicorn.run(app, host="0.0.0.0", port=8000)
