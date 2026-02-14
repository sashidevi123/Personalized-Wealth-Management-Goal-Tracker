main.py
from fastapi import FastAPI, Depends
from pydantic import BaseModel
from sqlalchemy.orm import Session
from fastapi.middleware.cors import CORSMiddleware


from database import Base, engine, SessionLocal
from models import User
from auth import hash_password, verify_password
from risk import calculate_risk

Base.metadata.create_all(bind=engine)

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# ---------- Schemas ----------
class SignupRequest(BaseModel):
    email: str
    password: str

class LoginRequest(BaseModel):
    email: str
    password: str

class RiskRequest(BaseModel):
    age: int
    salary: int
    investment: int

# ---------- DB Dependency ----------
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# ---------- Signup ----------
@app.post("/signup")
def signup(request: SignupRequest, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == request.email).first()
    if user:
        return {"error": "User already exists"}
    new_user = User(
        email=request.email,
        password=hash_password(request.password)
    )
    db.add(new_user)
    db.commit()
    return {"message": "Signup successful"}

# ---------- Login ----------
@app.post("/login")
def login(request: LoginRequest, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.email == request.email).first()
    if not user or not verify_password(request.password, user.password):
        return {"error": "Invalid credentials"}
    return {"message": "Login successful"}

# ---------- Risk ----------
@app.post("/risk")
def risk(request: RiskRequest):
    result = calculate_risk(
        request.age, request.salary, request.investment
    )
    return {"risk": result}







App.js

import React, { useState } from "react";

function App() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [message, setMessage] = useState("");

  const handleSignup = async () => {
    const response = await fetch("http://127.0.0.1:8000/signup", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ email, password }),
    });

    const data = await response.json();
    setMessage(JSON.stringify(data));
  };

  return (
    <div style={{ padding: "50px" }}>
      <h2>Signup</h2>
      <input
        type="email"
        placeholder="Enter Email"
        onChange={(e) => setEmail(e.target.value)}
      />
      <br /><br />
      <input
        type="password"
        placeholder="Enter Password"
        onChange={(e) => setPassword(e.target.value)}
      />
      <br /><br />
      <button onClick={handleSignup}>Signup</button>
      <p>{message}</p>
    </div>
  );
}

export default App;







database.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = "sqlite:///./users.db"

engine = create_engine(
    DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(bind=engine)
Base = declarative_base()





models.py

from sqlalchemy import Column, Integer, String
from database import Base

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    password = Column(String)





auth.py

from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"])

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(password: str, hashed: str):
    return pwd_context.verify(password, hashed)








risk.py

def calculate_risk(age, salary, investment):
    score = 0
    if age < 30:
        score += 3
    elif age < 50:
        score += 2
    else:
        score += 1
    if salary > 50000:
        score += 3
    elif salary > 30000:
        score += 2
    else:
        score += 1
    if investment > 200000:
        score += 3
    elif investment > 100000:
        score += 2
    else:
        score += 1
    if score <= 4:
        return "Low Risk"
    elif score <= 6:
        return "Medium Risk"
    else:
        return "High Risk"






output:

http://127.0.0.1:8000/docs#/



