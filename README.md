from decimal import Decimal
from uuid import uuid4

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field


app = FastAPI(title="CashBet API")

wallets: dict[str, Decimal] = {}
transactions: list[dict] = []

MIN_DEPOSIT = Decimal("10")
MIN_BET = Decimal("1")


class DepositRequest(BaseModel):
    user_id: str
    amount: Decimal = Field(gt=0)
    method: str


class BetRequest(BaseModel):
    user_id: str
    amount: Decimal = Field(gt=0)
    event: str
    odds: Decimal = Field(gt=0)


class WithdrawRequest(BaseModel):
    user_id: str
    amount: Decimal = Field(gt=0)


@app.post("/deposit")
def deposit(req: DepositRequest):

    if req.amount < MIN_DEPOSIT:
        raise HTTPException(
            status_code=400,
            detail=f"Minimum deposit is ${MIN_DEPOSIT}"
        )

    if req.method not in {"card", "crypto"}:
        raise HTTPException(
            status_code=400,
            detail="Method must be 'card' or 'crypto'"
        )

    wallets[req.user_id] = (
        wallets.get(req.user_id, Decimal("0"))
        + req.amount
    )

    tx = {
        "id": str(uuid4()),
        "type": "deposit",
        "user_id": req.user_id,
        "amount": req.amount,
        "method": req.method,
    }

    transactions.append(tx)

    return {
        "balance": wallets[req.user_id],
        "transaction": tx,
    }


@app.post("/bet")
def place_bet(req: BetRequest):

    bal = wallets.get(req.user_id, Decimal("0"))

    if req.amount > bal:
        raise HTTPException(
            status_code=400,
            detail="Insufficient balance"
        )

    if req.amount < MIN_BET:
        raise HTTPException(
            status_code=400,
            detail=f"Minimum bet is ${MIN_BET}"
        )

    wallets[req.user_id] = bal - req.amount

    tx = {
        "id": str(uuid4()),
        "type": "bet",
        "user_id": req.user_id,
        "amount": req.amount,
        "event": req.event,
        "odds": req.odds,
    }

    transactions.append(tx)

    return {
        "balance": wallets[req.user_id],
        "transaction": tx,
    }


@app.post("/withdraw")
def withdraw(req: WithdrawRequest):

    bal = wallets.get(req.user_id, Decimal("0"))

    if req.amount > bal:
        raise HTTPException(
            status_code=400,
            detail="Insufficient balance"
        )

    wallets[req.user_id] = bal - req.amount

    tx = {
        "id": str(uuid4()),
        "type": "withdraw",
        "user_id": req.user_id,
        "amount": req.amount,
    }

    transactions.append(tx)

    return {
        "balance": wallets[req.user_id],
        "transaction": tx,
    }


@app.get("/balance/{user_id}")
def get_balance(user_id: str):
    return {
        "balance": wallets.get(user_id, Decimal("0"))
    }


@app.get("/transactions/{user_id}")
def get_transactions(user_id: str):
    return [
        tx for tx in transactions
        if tx["user_id"] == user_id
    ]


@app.get("/health")
def health():
    return {"status": "ok"}
