# cashbet
CashBet is a betting and trading app
app = FastAPI(title=“CashBet API”)
wallets = transactions = [] MIN_DEPOSIT = 10
class DepositRequest(BaseModel): user_id: str amount: Decimal method: str
class BetRequest(BaseModel): user_id: str amount: Decimal event: str odds: Decimal
class WithdrawRequest(BaseModel): user_id: str amount: Decimal
@app.post(”/deposit”) def deposit(req: DepositRequest): if req.amount < MIN_DEPOSIT: raise HTTPException(400, f”Minimum deposit is ${MIN_DEPOSIT}”) if req.method not in (“card”, “crypto”): raise HTTPException(400, “Method must be ‘card’ or ‘crypto’”) wallets = wallets.get(req.user_id, 0) + req.amount tx = {“id”: str(uuid.uuid4()), “type”: “deposit”, “user_id”: req.user_id, “amount”: req.amount, “method”: req.method} transactions.append(tx) return {“balance”: wallets , “transaction”: tx}
@app.post(”/bet”) def place_bet(req: BetRequest): bal = wallets.get(req.user_id, 0) if req.amount > bal: raise HTTPException(400
, “Insufficient balance”) if req.amount < 1: raise HTTPException(400, “Minimum bet is $1”) wallets = bal - req.amount tx = {“id”: str(uuid.uuid4()), “type”: “bet”, “user_id”: req.user_id, “amount”: req.amount, “event”: req.event, “odds”: req.odds} transactions.append(tx) return {“balance”: wallets , “transaction”: tx}
@app.post(”/withdraw”) def withdraw(req: WithdrawRequest): bal = wallets.get(req.user_id, 0) if req.amount > bal: raise HTTPException(400, “Insufficient balance”) wallets = bal - req.amount tx = {“id”: str(uuid.uuid4()), “type”: “withdraw”, “user_id”: req.user_id, “amount”: req.amount} transactions.append(tx) return {“balance”: wallets , “transaction”: tx}
@app.get(”/balance/{user_id}”) def get_balance(user_id: str): return {“balance”: wallets.get(user_id, 0)}
@app.get(”/transactions/{user_id}”) def get_transactions(user_id: str): return [t for t in transactions if t == user_id]
@app.get(”/health”) def health(): return {“status”: “ok”}
