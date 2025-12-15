#!/usr/bin/env python3
"""
Script de démarrage simplifié pour SGA Web
"""

import subprocess
import sys
import os

def check_dependencies():
    """Vérifie que toutes les dépendances sont installées"""
    try:
        import fastapi
        import uvicorn
        import pandas
        print("✅ Toutes les dépendances sont installées")
        return True
    except ImportError as e:
        print(f"❌ Dépendance manquante: {e}")
        print("📦 Installation des dépendances...")
        try:
            subprocess.check_call([sys.executable, "-m", "pip", "install", "-r", "requirements.txt"])
            return True
        except subprocess.CalledProcessError:
            print("❌ Échec de l'installation des dépendances")
            return False

def main():
    """Point d'entrée principal"""
    print("\n" + "="*60)
    print("🚀 SGA Web - Système de Gestion des Agents")
    print("="*60)
    
    # Vérifier les dépendances
    if not check_dependencies():
        sys.exit(1)
    
    # Vérifier la base de données
    if not os.path.exists("database/planning.db"):
        print("⚠️  Base de données non trouvée")
        print("📁 Placez votre fichier planning.db dans le dossier database/")
        response = input("Créer une base vide ? (o/n): ")
        if response.lower() == 'o':
            # Initialiser une base vide
            from backend.main import init_database
            init_database()
            print("✅ Base de données vide créée")
        else:
            print("❌ L'application nécessite une base de données")
            sys.exit(1)
    
    # Démarrer l'API
    print("\n🌐 Démarrage de l'API...")
    print("📊 Accédez à: http://localhost:8000")
    print("📚 Documentation: http://localhost:8000/docs")
    print("\n🛑 Appuyez sur Ctrl+C pour arrêter\n")
    
    try:
        subprocess.run([
            sys.executable, "-m", "uvicorn", 
            "backend.main:app",
            "--host", "0.0.0.0",
            "--port", "8000",
            "--reload"
        ])
    except KeyboardInterrupt:
        print("\n\n👋 Arrêt de l'application")
    except Exception as e:
        print(f"❌ Erreur: {e}")

if __name__ == "__main__":
    main()
