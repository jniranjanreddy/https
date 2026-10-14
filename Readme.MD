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
## Testing..

```
4. Test it
Easiest way: open http://127.0.0.1:8000/docs in your browser. For any endpoint, click Try it out and then Execute, and you’ll see the status code.

Using curl: open a second terminal. In Windows PowerShell, type curl.exe, because plain curl runs a different PowerShell command. -i shows the status line in the output.

Code	What triggers it	Command
200	Item exists	curl.exe -i http://127.0.0.1:8000/items/1
404	Item doesn’t exist	curl.exe -i http://127.0.0.1:8000/items/99
400	Dividing by zero	curl.exe -i "http://127.0.0.1:8000/divide?a=5&b=0"
422	Wrong type (handled by FastAPI)	curl.exe -i "http://127.0.0.1:8000/divide?a=5&b=abc"
401	No token sent	curl.exe -i http://127.0.0.1:8000/admin
403	Wrong token	curl.exe -i -H "token: wrong" http://127.0.0.1:8000/admin
200	Correct token	curl.exe -i -H "token: admin-secret" http://127.0.0.1:8000/admin
405	Wrong HTTP method (handled by FastAPI)	curl.exe -i -X PUT http://127.0.0.1:8000/divide
307	Redirect	curl.exe -i http://127.0.0.1:8000/old-page
500	Server crash	curl.exe -i http://127.0.0.1:8000/crash
503	Maintenance	curl.exe -i http://127.0.0.1:8000/maintenance
204	Delete succeeded	curl.exe -i -X DELETE http://127.0.0.1:8000/items/2
Two more to try:

201 Created: curl.exe -i -X POST http://127.0.0.1:8000/items/5 -H "Content-Type: application/json" -d "{\"name\":\"mango\",\"price\":10}"
409 Conflict: run that same command again.
5. Things to notice
2xx means success, 3xx a redirect, 4xx the client made a mistake, and 5xx the server failed.
FastAPI returns 422 and 405 by itself; you didn’t write any code for them.
On /crash, check the server terminal and you’ll see the full error traceback. That’s the difference between a 500 and a 4xx you raised on purpose.
If you want, I can create these files and start the server for you in the Terminal pane.
```
```
curl.exe -i http://127.0.0.1:8000/items/1
curl.exe -i http://127.0.0.1:8000/items/99
curl.exe -i "http://127.0.0.1:8000/divide?a=5&b=0"
curl.exe -i "http://127.0.0.1:8000/divide?a=5&b=abc"
curl.exe -i http://127.0.0.1:8000/admin
curl.exe -i -H "token: wrong" http://127.0.0.1:8000/admin
curl.exe -i -H "token: admin-secret" http://127.0.0.1:8000/admin
curl.exe -i -X PUT http://127.0.0.1:8000/divide
curl.exe -i http://127.0.0.1:8000/old-page
curl.exe -i http://127.0.0.1:8000/crash
curl.exe -i http://127.0.0.1:8000/maintenance
curl.exe -i -X DELETE http://127.0.0.1:8000/items/2

curl -i http://127.0.0.1:8000/items/1
curl -i http://127.0.0.1:8000/items/99
curl -i "http://127.0.0.1:8000/divide?a=5&b=0"
curl -i "http://127.0.0.1:8000/divide?a=5&b=abc"
curl -i http://127.0.0.1:8000/admin
curl -i -H "token: wrong" http://127.0.0.1:8000/admin
curl -i -H "token: admin-secret" http://127.0.0.1:8000/admin
curl -i -X PUT http://127.0.0.1:8000/divide
curl -i http://127.0.0.1:8000/old-page
curl -i http://127.0.0.1:8000/crash
curl -i http://127.0.0.1:8000/maintenance
curl -i -X DELETE http://127.0.0.1:8000/items/2```
