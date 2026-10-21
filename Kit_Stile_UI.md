# Kit di stile — Guida al layout

Questa guida spiega come ricostruire lo stesso aspetto della dashboard
"Pianificazione Collaudi" in una nuova app. Il colore e i font stanno tutti
in `theme.py` (copia `theme_template.py` e rinominalo `theme.py`): per
ricolorare un'app cambi solo il blocco `ACCENT` in cima.

Sotto trovi gli snippet dei quattro elementi ricorrenti. Sono pensati per
essere copiati e adattati.

---

## 0. Struttura generale

Ogni finestra segue lo stesso schema a tre fasce impilate:

    ┌────────────────────────────────┐
    │  HEADER colorato (titolo)      │  ← fascia ACCENT in alto
    ├────────────────────────────────┤
    │                                │
    │  AREA CENTRALE (tabella,       │  ← contenuto, sfondo BG_APP
    │  form, ecc.)                   │
    │                                │
    ├────────────────────────────────┤
    │  BARRA PULSANTI + stato        │  ← pulsanti a sx, stato a dx
    └────────────────────────────────┘

Finestra base:

```python
import tkinter as tk
from tkinter import ttk
from theme import *   # oppure importa i singoli nomi

class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("La mia app")
        self.configure(bg=BG_APP)
        self._build_style()
        self._build_ui()
```

---

## 1. Stile della tabella (ttk)

Da chiamare una volta all'avvio. Usa il tema `clam` (l'unico che rispetta
bene i colori personalizzati) e imposta tabella, intestazioni e scrollbar.

```python
def _build_style(self):
    s = ttk.Style(self)
    s.theme_use("clam")
    s.configure("Treeview",
                background=BG_WHITE, fieldbackground=BG_WHITE,
                foreground=TEXT_MAIN, rowheight=24,
                font=FONT_LABEL, borderwidth=0)
    s.configure("Treeview.Heading",
                background=BG_THEAD, foreground=TEXT_SEC,
                font=FONT_BOLD, relief="flat", padding=4)
    s.map("Treeview.Heading", background=[("active", BORDER)])
    s.map("Treeview",
          background=[("selected", ACCENT)],
          foreground=[("selected", TEXT_WHITE)])
    for orient in ("Vertical.TScrollbar", "Horizontal.TScrollbar"):
        s.configure(orient, background=BORDER, troughcolor=BG_APP,
                    arrowcolor=TEXT_SEC, borderwidth=0)
```

---

## 2. Header colorato

Una fascia `ACCENT` in alto, altezza fissa, col titolo a sinistra. Il
`pack_propagate(False)` impedisce che la fascia si restringa al contenuto.

```python
hdr = tk.Frame(self, bg=ACCENT, height=HEADER_H)
hdr.pack(fill="x")
hdr.pack_propagate(False)
tk.Label(hdr, text="La mia app", bg=ACCENT, fg=TEXT_WHITE,
         font=FONT_TITLE).pack(side="left", padx=16)
# eventuale sottotitolo/badge a destra, in tinta chiara:
tk.Label(hdr, text="v1.0", bg=ACCENT, fg=ACCENT_LIGHT,
         font=FONT_LABEL).pack(side="right", padx=16)
```

---

## 3. Tabella con righe alternate (zebra)

La tabella vera e propria, dentro una cornice con bordo, con scrollbar
verticale. Le righe alterne si ottengono con due tag di colore.

```python
wrap = tk.Frame(self, bg=BG_WHITE, highlightbackground=BORDER,
                highlightthickness=1)
wrap.pack(fill="both", expand=True, padx=PAD, pady=PAD)

cols = ("nome", "valore", "note")
tree = ttk.Treeview(wrap, columns=cols, show="headings")
for c, w in (("nome", 160), ("valore", 120), ("note", 200)):
    tree.heading(c, text=c.capitalize())
    tree.column(c, width=w, stretch=True)

vsb = ttk.Scrollbar(wrap, orient="vertical", command=tree.yview)
tree.configure(yscrollcommand=vsb.set)
tree.pack(side="left", fill="both", expand=True)
vsb.pack(side="right", fill="y")

# Righe alterne: bianco / grigio chiarissimo
tree.tag_configure("odd", background=BG_WHITE)
tree.tag_configure("even", background=BG_ROW_ALT)
for i, riga in enumerate(dati):
    tree.insert("", "end", values=riga,
                tags=("even" if i % 2 else "odd",))
```

Per **colorare il testo in base a uno stato** (come la colonna status),
crea un tag per ogni stato e assegnalo alla riga:

```python
tree.tag_configure("st_ok",      foreground=CLR_ST_OK)
tree.tag_configure("st_pending", foreground=CLR_ST_PENDING)
# poi, inserendo: tags=("st_ok",)  oppure ("st_pending",)
```

---

## 4. Barra pulsanti in basso + stato

Pulsanti a sinistra, etichetta di stato a destra. Il pulsante primario usa
`ACCENT`; gli altri sono chiari con bordo.

```python
foot = tk.Frame(self, bg=BG_APP)
foot.pack(fill="x", padx=PAD, pady=PAD_SMALL)

# Pulsante primario (azione principale)
tk.Button(foot, text="Azione", bg=ACCENT, fg=TEXT_WHITE, relief="flat",
          font=FONT_BOLD, activebackground=ACCENT_MID,
          activeforeground=TEXT_WHITE, cursor="hand2",
          padx=14, pady=5).pack(side="left")

# Pulsante secondario (chiaro, con bordo)
tk.Button(foot, text="Altro", bg=BG_WHITE, fg=TEXT_MAIN, relief="solid",
          borderwidth=1, font=FONT_LABEL, activebackground=BG_THEAD,
          cursor="hand2", padx=12, pady=5).pack(side="left", padx=8)

# Etichetta di stato a destra
self.lbl_status = tk.Label(foot, text="Pronto", bg=BG_APP, fg=TEXT_SEC,
                           font=FONT_SMALL)
self.lbl_status.pack(side="right")
```

---

## 5. Titolo di sezione (per pannelli/form)

Nei pannelli di dettaglio, ogni sezione ha un'intestazione su fascia chiara
con testo in `ACCENT`:

```python
card = tk.Frame(parent, bg=BG_WHITE, highlightbackground=BORDER,
                highlightthickness=1)
card.pack(fill="x", padx=PAD, pady=(10, 0))
tk.Label(card, text="Titolo sezione", bg=BG_THEAD, fg=ACCENT,
         font=FONT_BOLD, anchor="w").pack(fill="x", ipady=3, ipadx=8)
```

---

## Riepilogo: le regole del "look"

1. **Un solo accento.** Header, pulsante primario, titoli di sezione e
   selezione della tabella usano `ACCENT`. Tutto il resto è neutro.
2. **Superfici bianche su sfondo grigio.** Contenuti (tabelle, card) su
   `BG_WHITE`, dentro cornici `BORDER`, sopra una finestra `BG_APP`.
3. **Righe a zebra** (`BG_WHITE` / `BG_ROW_ALT`) per leggibilità.
4. **Un font solo** (Segoe UI) in quattro misure: titolo, grassetto,
   normale, piccolo.
5. **Pulsante primario pieno** (accent), **secondari chiari** con bordo.
6. **Stato in basso a destra**, testo piccolo e grigio.

Segui questi sei punti e qualsiasi nuova app avrà lo stesso aspetto.
```