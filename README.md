
---

# 10 — SQL Migration Risk Scorer (migration_risk.py)

**File:** `src/migration_risk.py`
```python
#!/usr/bin/env python3
"""
SQL Migration Risk Scorer (prototype)
- Compares sample query outputs and response times between two schemas (simulated).
Run: python3 src/migration_risk.py
"""
import random, time, json

def simulate_query_workload(n=100):
    queries = []
    for i in range(n):
        q = {"id":i, "text": "SELECT * FROM users WHERE id=%d"%random.randint(1,1000)}
        queries.append(q)
    return queries

def run_against_schema(queries, schema_bias=1.0):
    results = []
    for q in queries:
        start = time.time()
        # simulate cost influenced by schema_bias
        time.sleep(0.001 * random.random() * schema_bias)
        rows = random.randint(0,5) if random.random()>0.1 else 0
        results.append({"id": q["id"], "rows": rows, "ms": (time.time()-start)*1000})
    return results

def score(old_res, new_res):
    diffs = []
    for o,n in zip(old_res,new_res):
        diffs.append(abs(o["rows"]-n["rows"]))
    avg_diff = sum(diffs)/len(diffs)
    avg_old = sum(r["ms"] for r in old_res)/len(old_res)
    avg_new = sum(r["ms"] for r in new_res)/len(new_res)
    return {"avg_row_diff": avg_diff, "old_ms":avg_old, "new_ms":avg_new}

def demo():
    qs = simulate_query_workload(200)
    old = run_against_schema(qs, schema_bias=1.0)
    new = run_against_schema(qs, schema_bias=1.2)
    print(json.dumps(score(old,new), indent=2))

if __name__ == "__main__":
    demo()
