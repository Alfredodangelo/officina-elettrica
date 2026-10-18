# Formato dei file `.cad` di CADe SIMU

Documentazione del formato di salvataggio di **CADe SIMU** (schemi elettrici), ricavata
per **interoperabilità e studio** analizzando l'eseguibile `CADe_SIMU.exe` con REA/Ghidra
e confrontando schemi salvati reali. Serve a leggere e ridisegnare i `.cad` senza il
programma originale (vedi il *Lettore CADe SIMU* in questa stessa app).

> CADe SIMU è freeware di **CANALIS**. Qui non è riprodotto alcun codice del programma:
> si documenta solo il formato dei dati, per poter aprire i propri file su altri sistemi.
> Codici e layout sono ricavati per reverse engineering: accurati ma non ufficiali.

---

## 1. Struttura generale

Il file è **testo semplice** (una sola riga, codifica latin-1). Tre sezioni separate dai
marcatori `$$$` e `&&&`:

```
CADe_SIMU*0*<versione>#<record componente>#<record componente>#...$$$<piè di pagina>&&&...
```

- **Intestazione**: inizia con la firma `CADe_SIMU`, seguita da `*0*<versione>` (es. `3005`).
- **Corpo**: una sequenza di **record**, uno per componente o filo, ciascuno introdotto da
  `*<id>*<tipo>#` e terminato da `#`.
- **`$$$`** chiude l'elenco dei componenti.
- **Piè di pagina** (dopo `$$$`, dentro i blocchi `&&&`): dimensione del foglio (es. `279*210`),
  data (es. `19-May-2022`), nome del disegno e alcuni flag.

### Separatori
| Simbolo | Significato |
|---|---|
| `*` | separatore di campo |
| `#` | separatore dei campi testo (sigla e morsetti) |
| `$$$` | fine dei componenti |
| `&&&` | separatore delle sezioni del piè di pagina |

---

## 2. Record di un componente

```
*<id>*<tipo>#<sigla>#<m1>#<m2>#…##<flag…>*…*<x>*<y>*…*<bbox…>*…*<nodoA>*<nodoB>#
```

Dopo `*<id>*<tipo>#`:

1. **sigla** del componente (es. `-KM1`), fino al primo `#`.
2. **nomi dei morsetti** (es. `1#3#5#2#4#6`), racchiusi tra due `##`.
3. una serie di **numeri** separati da `*`.

Interpretazione dei numeri (indici 0-based, ricavati; i più stabili sono quelli di posizione):

| indici | contenuto |
|---|---|
| 0…5 (o 0…7) | flag "morsetto presente" (6 per i tripolari, 8 per i quadripolari…) |
| 8, 9 | **x, y** del punto d'origine (morsetto in alto) |
| 12…15 | **riquadro** relativo `dx1, dy1, dx2, dy2` (ingombro del simbolo) |
| ultimi due | **nodi elettrici** collegati ai due morsetti principali |

### Fili e nodi (tipo 4000 / 4001 / 4010)
Per i fili gli indici **8…11** sono gli estremi del segmento: **x1, y1, x2, y2**.

### Due varianti di record
Esistono due formati, con **10** oppure **11** campi testo (`%s`). La variante a 11 campi è
usata dai tipi con un parametro/morsetto in più (p. es. intervallo `10499–10550` e alcuni
tipi isolati). Un lettore robusto accetta entrambe e conta i campi dinamicamente.

Le coordinate sono in piccole unità (griglia ~3 px); conviene adattarle al riquadro del disegno.

---

## 3. Tabella dei codici-tipo

Codici estratti dalle funzioni di disegno di `CADe_SIMU.exe` (globale "tipo corrente"
`DAT_00bd95cc`) e confermati su file reali. La colonna *famiglia* è il simbolo usato dal lettore.

### Motori e azionamenti
| codice | componente | famiglia |
|---|---|---|
| 1000 | motore | motore |
| 1003 | motore Dahlander | motore |
| 1004 | motore a rotore avvolto | motore |
| 1005 | motore a 2 velocità | motore |
| 1006 | motore cc magnete permanente | motore |
| 1007 | motore cc eccitazione indipendente | motore |
| 1050 | resistenza | resistenza |
| 1051 | resistenza III | resistenza |
| 1052 | autotrasformatore | trasformatore |
| 1054 | variatore velocità ca II | blocco |
| 1055 | variatore velocità ca III | blocco |
| 1057 | variatore velocità cc | blocco |
| 1058 | variatore velocità cc | blocco |
| 1059 | controllo variatore cc | blocco |

### Contattori, interruttori di manovra, trasformatori
| codice | componente | famiglia |
|---|---|---|
| 2000 | contattore 2 poli | potenza |
| 2001 | contattore 3 poli | potenza |
| 2002 | contattore 4 poli | potenza |
| 2006 | contattore 1 polo | potenza |
| 2003 | interruttore 2 poli | potenza |
| 2004 | interruttore/sezionatore 3 poli | potenza |
| 2005 | interruttore 4 poli | potenza |
| 2007 | interruttore 1 polo | potenza |
| 3012 | trasformatore | trasformatore |
| 3013 | raddrizzatore | blocco |

### Cavi, morsetti, connettori
| codice | componente | famiglia |
|---|---|---|
| 4000 | filo di fase | filo |
| 4001 | nodo/giunzione | nodo |
| 4002 | cavo multipolare | filo |
| 4003 / 4004 | entrata cavo | filo |
| 4009 | filo di neutro | filo |
| 4010 | filo di terra (PE) | filo |
| 4018 / 4019 | filo + / filo − | filo |
| 4011 | morsettiera | morsetto |
| 4012 / 4015 / 4016 / 4017 | morsetto | morsetto |
| 4013 | connettore di uscita | morsetto |
| 4014 | connettore di entrata | morsetto |

### Fusibili, sezionatori, protezioni
| codice | componente | famiglia |
|---|---|---|
| 5000 | fusibile | fusibile |
| 5001 | fusibile 1P+N | fusibile |
| 5002 | fusibile 2P | fusibile |
| 5003 | fusibile 3P | fusibile |
| 5004 | fusibile 3P+N | fusibile |
| 6002 | fusibile / protezione | fusibile |
| 5005…5009 | sezionatore 1P / 1P+N / 2P / 3P / 3P+N | potenza |
| 6005 | differenziale 1P+N | potenza |
| 6006 | differenziale 3P+N | potenza |
| 6008 | magnetotermico 2P | potenza |
| 6009 | magnetotermico 3P | potenza |
| 6010 | magnetotermico 1P | potenza |
| 6015 | magnetotermico | potenza |
| 6018 | interruttore differenziale | potenza |
| 6019 | interruttore | potenza |
| 6020 | sezionatore | potenza |

### Contatti ausiliari
| codice | componente | famiglia |
|---|---|---|
| 7000 | contatto ausiliario NA | contatto NA |
| 7001 | contatto ausiliario NC | contatto NC |
| 7002 | ausiliario NA-NC | contatto |
| 7003 | ausiliario in scambio | contatto |
| 7004 | contatto di timer NA | contatto NA |
| 7018 | contatto di contattore | contatto |
| 7019 | contatti ausiliari | contatto |
| 7020 | contatto ritardato ON-delay | contatto NA |
| 7021 | contatto ritardato OFF-delay | contatto NC |

### Pulsanti, interruttori, finecorsa
| codice | componente | famiglia |
|---|---|---|
| 8000 | pulsante NA | pulsante NA |
| 8001 | pulsante NC | pulsante NC |
| 8002 | pulsante NA-NC | pulsante |
| 8003 | pulsante in scambio | pulsante |
| 8004 / 8005 / 8006 / 8007 | pulsante a fungo NA / NC / NA-NC / scambio | pulsante |
| 8008 / 8009 / 8010 / 8011 | contatto interruttore NA / NC / NA-NC / scambio | contatto |
| 8012 / 8013 / 8014 / 8015 | finecorsa NA / NC / NA-NC / scambio | contatto |
| 8016 | ausiliario di interruttore | contatto |
| 8100 | pulsante | pulsante |
| 8101 | spia luminosa | lampada |
| 8102 | pulsantiera | blocco |

### Bobine e segnalazioni
| codice | componente | famiglia |
|---|---|---|
| 9000 | bobina monostabile | bobina |
| 9001 | bobina bistabile | bobina |
| 9002 | telerruttore | bobina |
| 9004 | bobina di timer | bobina timer |
| 9007 | timer intermittente | bobina timer |
| 9009 | lampada | lampada |
| 9010 | campanello | segnalazione |
| 9011 | cicalino | segnalazione |
| 9012 | sirena | segnalazione |
| 9013 | tromba | segnalazione |
| 9016 | bobina di contatore | contatore |

### Logica e simulazione
| codice | componente | famiglia |
|---|---|---|
| 10050 | contatto NA | contatto NA |
| 10051 | contatto NC | contatto NC |
| 10052 | fronte di salita | contatto |
| 10053 | fronte di discesa | contatto |
| 10060 | assegna | bobina |
| 10061 | set ripetuto | bobina |
| 10062 | reset | bobina |
| 10063 | contatore di impulsi | contatore |
| 10064 | bistabile RS | blocco |
| 10065 | divisore binario | blocco |
| 10070 | prova | blocco |
| 10071 | visualizza | blocco |
| 10075 | contatore avanti-indietro | contatore |

### GRAFCET
| codice | componente | famiglia |
|---|---|---|
| 10100 | Etapa 0 (passo iniziale) | passo GRAFCET |
| 10101 | Etapa (passo) | passo GRAFCET |
| 10107 | enlace di entrata | legame GRAFCET |
| 10108 | enlace di uscita | legame GRAFCET |
| 10111 | macro-etapa | passo GRAFCET |
| 10112 | Y (divergenza AND) | legame GRAFCET |

### PLC (Siemens S7-1200 / S7-1500)
| codice | componente | famiglia |
|---|---|---|
| 10204 | ingresso digitale | PLC |
| 10205 | uscita digitale | PLC |
| 10206 | CPU | PLC |
| 10207 | alimentazione I/O | PLC |
| 10208 | alimentazione I/O | PLC |

### Marcatori
| codice | componente |
|---|---|
| 3005 | origine del foglio |
| 20000 | fine foglio |

---

## 4. Note

- I campi di coordinate (indici 8–9 per i componenti, 8–11 per i fili) sono verificati su file
  reali; gli altri indici sono interpretazioni e possono variare per alcuni tipi.
- Le connessioni elettriche "chi è collegato a chi" nei file reali si deducono dalla **geometria**
  (estremi dei fili coincidenti con i morsetti); gli ultimi due numeri del record sembrano
  riferirsi ai nodi, ma la ricostruzione completa della rete richiede più schemi di esempio.
- GRAFCET e PLC: i codici sono certi; il disegno dettagliato del simbolo migliora con un file
  di esempio che li contenga.

---

## 5. Esportazione da *Editor Schemi* (scrittura dei `.cad`)

L'*Editor Schemi* di questa app può **salvare** uno schema in formato `.cad`, così lo si
riapre in CADe SIMU. L'esportazione è il percorso inverso dell'import ed è **sperimentale**:
il file va provato aprendolo davvero in CADe SIMU, perché il formato è ricostruito, non ufficiale.

I record generati sono stati **confrontati con file `.cad` reali** salvati da CADe SIMU
(esercizi scolastici): contatti (7000/7001/7004), pulsanti (8000/8001), bobine (9000/9004),
contattore (2001) e trasformatore (3012) risultano **identici** a quelli reali, a meno di
id, coordinate, nodi e dell'eventuale descrizione del componente.

### Cosa viene scritto
1. **Intestazione**: il record origine del foglio
   `CADe_SIMU*0*3005#-X#…#` (tipo 3005), con `id = 0`.
2. **Un record per componente**, con `id` progressivo da 1:

   ```
   *<id>*<tipo>#-<sigla>##<m1>#<m2>#…##<flag…>*<x>*<y>*0*0*<bbox>*0*0*0*1*0*0*0#
   ```

   - il blocco testo ha sempre **10 campi** (`%s`): sigla **senza trattino** (come nei file
     reali — il trattino in CADe compare solo sulle sigle di default non rinominate, es. `-M`),
     un campo **descrizione** (vuoto se assente, es. `START`/`STOP` sui pulsanti), i nomi dei
     morsetti, poi riempimento con campi vuoti (variante standard, cfr. §2);
   - i **flag "morsetto presente"** e il **riquadro** (`bbox`) sono quelli tipici del tipo,
     ricavati da record reali;
   - i due **nodi** finali sono scritti a `0*0`: CADe SIMU ricostruisce la rete dalla
     geometria dei fili, quindi non servono per ridisegnare lo schema.
3. **Un record per ogni tratto di filo** (tipo `4000`), con gli estremi `x1,y1,x2,y2` nei
   campi 8–11:

   ```
   *<id>*4000##########*0*0*0*0*0*0*0*0*<x1>*<y1>*<x2>*<y2>*0*0*0*0*0*0*0*0*0*1*0#
   ```
   Una polilinea dell'editor viene spezzata in più segmenti, uno per record.
4. **Piè di pagina**: `$$$…&&&…` con foglio `279*210`, data corrente e nome del disegno.

### Conversione delle coordinate
Inversa dell'import: `unità_CADe = round(px_editor / 16 * 6)` (16 px = 1 cella dell'editor =
6 unità CADe). Il round-trip *editor → .cad → editor* conserva tipi, posizioni e sigle.

### Mappa tipo editor → codice CADe (export)
| editor | codice | | editor | codice |
|---|---|---|---|---|
| `no` | 7000 | | `k3` / `k4` | 2001 |
| `nc` | 7001 | | `sez3` | 2004 |
| `pno` / `pnoe` | 8000 | | `mt3` / `diff3` | 6009 |
| `pnc` | 8001 | | `fuse` | 5000 |
| `tno` | 7004 | | `res` | 1050 |
| `coil` / `set` / `reset` | 9000 | | `motor` | 1000 |
| `coilt` | 9004 | | `transf` | 3012 |
| `lamp` / `horn` | 9009 | | `edgep` → 7000, `edgen` → 7001 | |
| `cnt` | 9016 | | | |

### Limiti noti
- **GRAFCET, PLC e la sorgente L/N** (`gstep`, `ginit`, `glink`, `plci`, `plco`, `plccpu`,
  `src`) **non** vengono esportati: non hanno un record affidabile. Il numero di elementi
  saltati è mostrato a fine esportazione.
- Alcune corrispondenze sono **approssimazioni**: il contattore 4 poli esce come 3 poli
  (2001), la sirena come lampada (9009), il pulsante d'emergenza come pulsante normale (8000).
- La **rotazione** dei componenti non viene riportata: in CADe escono in verticale
  (i fili, scritti con le coordinate reali, restano al posto giusto).
- Il `bbox` serve solo alla selezione in CADe e può non essere perfetto: non impedisce il
  disegno del simbolo, guidato dal codice-tipo.

---

*Documentazione generata nel contesto del progetto Officina Elettrica. Reverse engineering a
scopo di interoperabilità e didattica.*
