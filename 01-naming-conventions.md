# 📋 ABAP Naming Conventions

## Principi Generali

- **Namespace**: Tutti gli oggetti custom devono iniziare con `Z` o `Y`
- **Descrittivo**: I nomi devono essere auto-esplicativi
- **Consistenza**: Mantieni la stessa convenzione all'interno di un progetto
- **Inglese**: Usa l'inglese per nomi e commenti
- **No Caratteri Speciali**: Evita `/`, spazi, caratteri non-ASCII

---

## Naming per Tipo di Oggetto

### Programmi (ABAP Reports)

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Report (List) | `ZRP_` | `ZRP_ORDER_LIST` | Report con spool output |
| Report (Interactive) | `ZRP_` | `ZRP_ORDER_INTERACTIVE` | Report con user interaction |
| Program (Batch) | `ZPG_` | `ZPG_BATCH_PROCESSOR` | Background job |
| Program (Online) | `ZPG_` | `ZPG_CUSTOMER_MAINT` | Online transaction-like |

**Convenzione**: `Z` + `R` (Report) o `P` (Program) + `_` + DESCRIZIONE_SIGNIFICATIVA

---

### Function Module & Function Group

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Function | `Z_` o `ZFM_` | `Z_GET_CUSTOMER_DATA` | Funzione singola |
| Function Group | `ZFG_` | `ZFG_BILLING` | Raggruppamento logico |

**Convenzione**: 
- Funzione: `Z_` + VERBO + OGGETTO (es. `Z_CREATE_ORDER`, `Z_VALIDATE_ITEM`)
- FG: `ZFG_` + DOMINIO (es. `ZFG_SALES`, `ZFG_LOGISTICS`)

**Esempio di naming coerente**:
```
Function Group: ZFG_BILLING
  ├─ Z_GET_BILLING_DATA
  ├─ Z_CREATE_INVOICE
  ├─ Z_UPDATE_INVOICE
  └─ Z_DELETE_INVOICE
```

---

### Classi OOP

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Classe | `ZCL_` | `ZCL_SALES_ORDER` | Business logic class |
| Interface | `ZIF_` | `ZIF_PAYMENT_PROCESSOR` | Contract definition |
| Exception Class | `ZCX_` | `ZCX_INVALID_ORDER` | Custom exception |

**Convenzione Naming**:
- Singular nouns: `ZCL_CUSTOMER` (non `ZCL_CUSTOMERS`)
- PascalCase in metodi: `get_customer_data()`, `create_order()`
- Private vs Public: `_private_method`, `public_method`

**Esempio di struttura classe**:
```abap
CLASS ZCL_SALES_ORDER DEFINITION.
  PUBLIC SECTION.
    METHODS: create_order IMPORTING iv_customer TYPE vbeln,
             get_order_items RETURNING VALUE(rt_items) TYPE tt_items.
  PRIVATE SECTION.
    DATA: mv_order_id TYPE vbeln.
    METHODS: _validate_order.
ENDCLASS.
```

---

### Tabelle e Strutture

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Tabella Trasparente | `Z` | `ZCUSTOMER_DATA` | Database table |
| Tabella Temporanea | `ZT_` | `ZT_ORDER_ITEMS` | Work table |
| Struttura | `ZS_` | `ZS_BILLING_DOC` | Structure definition |
| View | `ZV_` | `ZV_CUSTOMER_ORDER` | View |

**Convenzione**: Singular, PascalCase se possibile

---

### Data Elements & Domains

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Data Element | `ZE_` | `ZE_AMOUNT` | Semantic type |
| Domain | `ZD_` | `ZD_STATUS` | Value domain |

---

### Enhancements & Modifications

| Tipo | Prefisso | Esempio | Note |
|------|----------|---------|-------|
| Enhancement (BADI) | `Z_` | `Z_ORDER_VALIDATION` | BADI implementation |
| User Exit | `Z_` | `Z_SD_PRESAVE_LOGIC` | User exit |
| Include | `ZINC_` | `ZINC_COMMON_ROUTINES` | Include program |

---

## Variabili - Naming Conventions

### Prefissi Locali

```abap
lv_name       " Local Variable (scalar)
lv_amount TYPE p DECIMALS 2

lt_items      " Local Table (internal table)
lt_orders TYPE TABLE OF zs_order

ls_item       " Local Structure (work area / line)
ls_item TYPE zs_order

lc_max_value  " Local Constant
CONSTANT lc_max_value TYPE i VALUE 1000.

lo_object     " Local Object (class instance)
lo_customer TYPE REF TO zcl_customer

<fs_field>    " Field Symbol
FIELD-SYMBOLS: <fs_item> TYPE zs_order.
```

### Prefissi Globali

```abap
gv_counter    " Global Variable
DATA: gv_counter TYPE i VALUE 0.

gt_buffer     " Global Table
DATA: gt_cache TYPE TABLE OF zs_item.

gs_config     " Global Structure
DATA: gs_config TYPE zs_config.

gc_version    " Global Constant (può essere anche 'c_')
CONSTANT gc_version TYPE string VALUE '1.0'.

go_instance   " Global Object
DATA: go_logger TYPE REF TO zcl_logger.
```

### Parametri di Subroutine/Metodo

```abap
p_customer    " Parameter (from selection screen)
PARAMETERS: p_customer TYPE kunnr.

p_from_date   " Parameter range
SELECT-OPTIONS s_date FOR sy-datum.

iv_id         " Importing parameter
METHOD process IMPORTING iv_order_id TYPE vbeln.

ev_result     " Exporting parameter
METHOD calculate EXPORTING ev_total TYPE p.

ic_data       " Importing (table/complex)
METHOD process IMPORTING ic_orders TYPE tt_orders.

ec_errors     " Exporting (table/complex)
METHOD validate EXPORTING ec_errors TYPE tt_messages.

cv_status     " Changing parameter
METHOD update_status CHANGING cv_status TYPE string.
```

---

## Convenzioni Specifiche

### Booleani
```abap
lv_is_active        " Boolean (is_, has_, can_)
lv_has_items TYPE abap_bool
lv_can_delete TYPE abap_bool
```

### Identificatori
```abap
lv_customer_id     " Customer ID
lv_order_num       " Order number
lv_line_item       " Line item counter
```

### Risultati/Codici Errore
```abap
lv_return_code     " Return code (0=success, non-0=error)
ls_return TYPE bapiret2  " Standard SAP return structure
```

### Quantità/Importi
```abap
lv_qty             " Quantity
lv_amount TYPE p DECIMALS 2
lv_percentage      " Percentage value
```

---

## Anti-Pattern (❌ Evita)

```abap
" ❌ Nomi troppo corti/criptici
DATA: x TYPE i.           " No!
DATA: tmp TYPE string.    " No!
DATA: var1, var2, var3.   " No!

" ❌ Nomi troppo lunghi
DATA: local_variable_that_contains_the_total_of_all_invoices_with_status_open TYPE p.

" ❌ Nomi non descrittivi
DATA: lv_data TYPE string.      " No! Quale data?
DATA: lv_result TYPE any.       " No! Quale risultato?
DATA: lv_temp TYPE table_type.  " No! Temporaneo è vago

" ❌ Mescolare prefissi
DATA: local_table TYPE TABLE OF items.  " No! Mix di stile

" ❌ Numeri invece di nomi significativi
DATA: lv_field1, lv_field2, lv_field3.  " No!
```

---

## Best Practices Riepilogo

✅ **Do's**
- Usa prefissi consistenti per tipo di variabile
- Nomi descrittivi (2-3 parole significative)
- Lowercase per variabili, PascalCase per classi
- Scegli `Z` come namespace (più standard)
- Usa verbi per azioni: `GET_`, `CREATE_`, `UPDATE_`, `DELETE_`
- Usa aggettivi per booleani: `IS_`, `HAS_`, `CAN_`

❌ **Don'ts**
- Abbreviazioni non standard (a meno che SAP standard)
- Nomi con numeri (v1, v2, field1)
- Caratteri speciali in nomi
- Nomi troppo generici (data, result, temp)
- Mescolare convenzioni nello stesso file

---

## Quick Lookup Table

| Elemento | Prefisso | Esempio | Stile |
|----------|----------|---------|-------|
| Local Var | `lv_` | `lv_total` | lowercase_underscore |
| Local Table | `lt_` | `lt_items` | lowercase_underscore |
| Local Struct | `ls_` | `ls_order` | lowercase_underscore |
| Global Var | `gv_` | `gv_counter` | lowercase_underscore |
| Constant | `lc_`, `c_` | `lc_max_value` | lowercase_underscore |
| Object | `lo_`, `go_` | `lo_customer` | lowercase_underscore |
| Class | `ZCL_` | `ZCL_ORDER` | PascalCase |
| Interface | `ZIF_` | `ZIF_VALIDATOR` | PascalCase |
| Function | `Z_` | `Z_CREATE_ORDER` | UPPERCASE_UNDERSCORE |
| Function Group | `ZFG_` | `ZFG_SALES` | PascalCase |
| Table | `Z` | `ZORDERS` | UPPERCASE |
| Structure | `ZS_` | `ZS_ORDER_DATA` | UPPERCASE_UNDERSCORE |

---

## Migrazione da Vecchi Standard

Se erediti codice vecchio con nomi non coerenti:

1. **Non rinominare** tutto immediatamente - causa rework inutile
2. **Nuovo codice** deve seguire le convenzioni di questo documento
3. **Refactoring**: Durante maintenance, aggiungi il naming corretto
4. **Documentazione**: Crea mapping di nomi vecchi → nuovi in commenti

---

**Ultimo aggiornamento**: Ottobre 2026
