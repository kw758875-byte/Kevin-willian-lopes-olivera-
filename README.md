# Kevin-willian-lopes-olivera-# ==================================================
# WIZZAPP - CORE FINAL ABSOLUTO
# ==================================================

import zlib

# ------------------ CONSTANTES ------------------
GB = 1024 * 1024 * 1024
X_TOTAL_GB = 2
X_TOTAL_BYTES = X_TOTAL_GB * GB

PLANOS = {
    "free": {
        "limite_memoria": int(X_TOTAL_BYTES * 0.7),
        "pastas_max": 3,
        "ordens_complexas": False
    },
    "premium": {
        "limite_memoria": X_TOTAL_BYTES,
        "pastas_max": 999,
        "ordens_complexas": True
    }
}

USUARIO = {"plano": "free"}

RAM_RATIO = 0.3
DISK_PATH = "wizzapp_storage.bin"

# ------------------ ESTADO ------------------
WIZZAPP = {
    "folders": {},
    "cloud": {},
    "logs": []
}

# ------------------ UTILITÁRIOS ------------------
def log(e): WIZZAPP["logs"].append(e)
def gerar_id(): return str(len(WIZZAPP["folders"]) + 1)
def tamanho_bytes(t): return len(t.encode("utf-8"))

def memoria_total():
    total = 0
    for p in WIZZAPP["folders"].values():
        for m in p["memoria"]:
            total += tamanho_bytes(m)
    return total

# ------------------ PERSONALIDADE ------------------
def gerar_personalidade(obj):
    obj = obj.lower()
    if "pesquisa" in obj: return "IA analítica e técnica"
    if "execução" in obj: return "IA orientada a ação"
    if "financeiro" in obj: return "IA estratégica"
    return f"IA focada em {obj}"

# ------------------ COMPRESSÃO ------------------
def comprimir(t): return zlib.compress(t.encode("utf-8"))
def salvar_disco(b):
    with open(DISK_PATH, "ab") as f: f.write(b + b"\n")

# ------------------ MEMÓRIA ------------------
def memorizar(pid, c):
    limite = PLANOS[USUARIO["plano"]]["limite_memoria"]
    while memoria_total() + tamanho_bytes(c) > limite:
        for p in WIZZAPP["folders"].values():
            if p["memoria"]:
                p["memoria"].pop(0)
                break
    if memoria_total() < limite * RAM_RATIO:
        WIZZAPP["folders"][pid]["memoria"].append(c)
    else:
        salvar_disco(comprimir(c))

# ------------------ PASTAS ------------------
def criar_pasta(n, o):
    if len(WIZZAPP["folders"]) >= PLANOS[USUARIO["plano"]]["pastas_max"]:
        return "Limite de pastas atingido"
    pid = gerar_id()
    WIZZAPP["folders"][pid] = {
        "nome": n,
        "objetivo": o,
        "personalidade": gerar_personalidade(o),
        "memoria": [],
        "links": []
    }
    log(f"Pasta criada: {n}")
    return pid

# ------------------ ORDENS ------------------
def classificar(t):
    for k in ["criar", "executar", "analisar", "comparar", "gerar"]:
        if k in t.lower(): return "complexa"
    return "simples"

def responder(pid, txt):
    p = WIZZAPP["folders"][pid]["personalidade"]
    return f"[{p}] {txt}"

# ------------------ EXECUÇÃO ------------------
def executar(pid, entrada):
    memorizar(pid, entrada)
    tipo = classificar(entrada)
    if tipo == "complexa" and not PLANOS[USUARIO["plano"]]["ordens_complexas"]:
        return "Recurso premium"
    resp = responder(pid, entrada)
    memorizar(pid, resp)
    return resp

# ------------------ STATUS ------------------
def status():
    usado = memoria_total()
    lim = PLANOS[USUARIO["plano"]]["limite_memoria"]
    return f"{round(usado/GB,2)}GB / {round(lim/GB,2)}GB"

# ------------------ EXECUÇÃO LOCAL ------------------
if __name__ == "__main__":
    p = criar_pasta("Pesquisa IA", "pesquisa técnica")
    print(executar(p, "analise este projeto"))
    print(status())
