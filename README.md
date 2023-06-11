#APi
"""from typing import List
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

origins = [
    "http://localhost:3000",  # Autoriser les requêtes CORS depuis l'application React
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

class User(BaseModel):
    email: str
    name: str
    uid: str


users = [
    {"email":"famienleroi.amoin@bnpparibas.com","name":"FamienLeRoiAMOIN","uid":"f45933"},
    {"email":"johndoe@example.com","name":"John Doe","uid":"f45934"},
    {"email":"janedoe@example.com","name":"Jane Doe","uid":"f45935"},
]  # Liste fictive d'utilisateurs


@app.get("/api/account/users/{uid}", response_model=List[User])
async def get_user_data(uid: str):
#@app.get("/api/account/users/{uid}", response_model=User)
#async def get_user_data(uid: str):
    for user in users:
        if user["uid"] == uid:
            #return user
            return [user for user in users if user["uid"] == uid]
    raise HTTPException(status_code=404, detail="Utilisateur non trouvé")
"""

from typing import List
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
import requests
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy import create_engine, Column, String
from sqlalchemy.orm import sessionmaker, Session
from sqlalchemy.ext.declarative import declarative_base

DATABASE_URL = "postgresql://postgres:0000@localhost:5433/projetFormation"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

class User(BaseModel):
    email: str
    name: str
    uid: str

class UserTable(Base):
    __tablename__ = "Utilisateur"

    email = Column(String, primary_key=True, index=True)
    name = Column(String)
    uid = Column(String)

app = FastAPI()

origins = [
    "http://localhost:3000",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/api/account/users/{uid}", response_model=List[User])
async def get_user_data(uid: str, db: Session = Depends(get_db)):
    response = requests.get(f'https://marketplace.staging.echonet/api/account/users?uids[]={uid}')
    data = response.json()

    if data:
        for user_data in data:
            user = UserTable(**user_data)
            db.add(user)
            db.commit()
            db.refresh(user)

        return data

    raise HTTPException(status_code=404, detail="Utilisateur non trouvé")



App.js:
import React, { useState } from "react";

function App() {
  const [uid, setUid] = useState("");
  const [userData, setUserData] = useState(null);

  const getUserData = async () => {
    const response = await fetch(`http://localhost:8000/api/account/users/${uid}`);
    const data = await response.json();
    setUserData(data);
  };

  return (
    <div className="App">
      <input
        type="text"
        value={uid}
        onChange={e => setUid(e.target.value)}
        placeholder="Entrez l'UID ici"
      />
      <button onClick={getUserData}>Obtenir les informations de l'utilisateur</button>
      {userData && (
        <div>
          <p>{JSON.stringify(userData)}</p>
        </div>
      )}
    </div>
  );
}

export default App;
