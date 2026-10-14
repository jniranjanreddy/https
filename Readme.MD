## http Error Codes.

```
mkdir http-practice
cd http-practice
python -m venv venv
venv\Scripts\activate
pip install fastapi uvicorn
```





##
```
from fastapi import FastAPI, HTTPException, Response, Header
from fastapi.responses import RedirectResponse
from pydantic import BaseModel

app = FastAPI()

items = {1: "apple", 2: "banana"}

class Item(BaseModel):
    name: str
    price: float

# 200 OK
@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")   # 404
    return {"id": item_id, "name": items[item_id]}

# 201 Created / 409 Conflict
@app.post("/items/{item_id}", status_code=201)
def create_item(item_id: int, item: Item):
    if item_id in items:
        raise HTTPException(status_code=409, detail="Item already exists")
    items[item_id] = item.name
    return {"id": item_id, "name": item.name}

# 204 No Content
@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    del items[item_id]
    return Response(status_code=204)

# 400 Bad Request
@app.get("/divide")
def divide(a: int, b: int):
    if b == 0:
        raise HTTPException(status_code=400, detail="b cannot be zero")
    return {"result": a / b}

# 401 Unauthorized / 403 Forbidden
@app.get("/admin")
def admin(token: str | None = Header(default=None)):
    if token is None:
        raise HTTPException(status_code=401, detail="Missing token")
    if token != "admin-secret":
        raise HTTPException(status_code=403, detail="Not allowed")
    return {"message": "Welcome admin"}

# 307 Redirect
@app.get("/old-page")
def old_page():
    return RedirectResponse(url="/items/1")

# 500 Internal Server Error (an unhandled crash)
@app.get("/crash")
def crash():
    return 1 / 0

# 503 Service Unavailable
@app.get("/maintenance")
def maintenance():
    raise HTTPException(status_code=503, detail="Under maintenance")
```
## execute uvicorn main:app --reload
