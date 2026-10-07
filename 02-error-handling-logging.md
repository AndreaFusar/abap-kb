# 🚨 Error Handling & Logging Best Practices

## Exception Handling - TRY...CATCH

### Principio Base

Preferisci **exception handling** rispetto ai return codes:

```abap
" ❌ OLD STYLE - Return Code
CALL FUNCTION 'Z_GET_CUSTOMER'
  EXPORTING
    iv_customer_id = lv_id
  IMPORTING
    ev_return_code = lv_rc
    es_customer = ls_customer.

IF lv_rc NE 0.
  " Gestisci errore
ENDIF.

" ✅ NEW STYLE - Exception
TRY.
    lo_customer = zcl_customer=>get_by_id( lv_id ).
  CATCH zcx_customer_not_found INTO DATA(lx_error).
    " Gestisci errore specifico
  CATCH cx_root INTO DATA(lx_generic).
    " Fallback generico
ENDTRY.
```

---

### Struttura Completa TRY...CATCH

```abap
TRY.
    " Logica principale
    lv_result = lo_processor->process_order( iv_order_id ).
    
  CATCH zcx_invalid_order INTO DATA(lx_invalid).
    " Eccezione specifica 1
    MESSAGE lx_invalid->get_text( ) TYPE 'E'.
    
  CATCH zcx_authorization_error INTO DATA(lx_auth).
    " Eccezione specifica 2 - Autorizzazione
    MESSAGE lx_auth->get_text( ) TYPE 'A'. " Abort
    
  CATCH cx_sy_arithmetic_error INTO DATA(lx_math).
    " Eccezione tecnica SAP
    WRITE: / 'Errore aritmetico:', lx_math->get_text().
    
  CATCH cx_root INTO DATA(lx_any).
    " Fallback - generico
    MESSAGE 'Errore sconosciuto' TYPE 'E'.
ENDTRY.
```

---

### Best Practices

✅ **Cattura specifico prima di generico**
```abap
TRY.
    " logica
  CATCH zcx_specific_error INTO DATA(lx_specific).
    " Gestisci specifico
  CATCH cx_root INTO DATA(lx_generic).
    " Fallback generico
ENDTRY.
```

✅ **Non ingoiare eccezioni senza loggare**
```abap
TRY.
    lo_processor->do_something( ).
  CATCH cx_root INTO DATA(lx_error).
    " SEMPRE loga l'errore!
    _add_log_message(
      iv_type = 'E'
      iv_message = lx_error->get_text( ) ).
    " Poi decidi: re-raise o continua
    RAISE EXCEPTION lx_error.
ENDTRY.
```

✅ **Cleanup in FINALLY**
```abap
TRY.
    lo_file->open( ).
    lo_file->process( ).
    
  CATCH cx_root INTO DATA(lx_error).
    MESSAGE lx_error->get_text( ) TYPE 'E'.
    
  FINALLY.
    " Cleanup SEMPRE eseguito
    lo_file->close( ).
ENDTRY.
```

---

## Logging con Application Log (BAL)

### Introduzione BAL

Il **BAL (Business Application Log)** è il framework standard di SAP per logging:
- Transazioni: `SLG1` (visualizza), `SLG0` (mantieni)
- Vantaggio: Centralizzato, filtrato, archivabile
- No codice custom di logging!

---

### Template Logging Completo

```abap
CLASS zcl_logger DEFINITION.
  PUBLIC SECTION.
    TYPES: BEGIN OF ty_message,
             type TYPE balmstype,      " 'E'=Error, 'W'=Warning, 'I'=Info, 'S'=Success
             message TYPE string,
             details TYPE string,
           END OF ty_message.
    
    METHODS: constructor IMPORTING iv_object TYPE balobjnr
                                    iv_subobject TYPE balsubnr,
             add_message IMPORTING is_message TYPE ty_message,
             save_and_display,
             get_log_handle RETURNING VALUE(rv_handle) TYPE balloghndl.
    
  PRIVATE SECTION.
    DATA: mv_log_handle TYPE balloghndl.
ENDCLASS.

CLASS zcl_logger IMPLEMENTATION.
  METHOD constructor.
    " Crea un nuovo log
    CALL FUNCTION 'BAL_LOG_CREATE'
      EXPORTING
        object     = iv_object
        subobject  = iv_subobject
        desc       = 'Application Processing Log'
      IMPORTING
        log_handle = mv_log_handle.
  ENDMETHOD.
  
  METHOD add_message.
    " Aggiungi messaggio al log
    CALL FUNCTION 'BAL_LOG_MSG_ADD'
      EXPORTING
        log_handle = mv_log_handle
        msg_type   = is_message-type
        msg_text   = is_message-message
        msg_var1   = is_message-details.
  ENDMETHOD.
  
  METHOD save_and_display.
    " Salva il log
    CALL FUNCTION 'BAL_DB_SAVE'
      EXPORTING
        log_handle = mv_log_handle.
    
    " Mostra il log all'utente
    CALL FUNCTION 'BAL_DSP_LOG_DISPLAY'
      EXPORTING
        i_log_handle = mv_log_handle.
  ENDMETHOD.
  
  METHOD get_log_handle.
    rv_handle = mv_log_handle.
  ENDMETHOD.
ENDCLASS.
```

---

### Uso del Logger

```abap
DATA: lo_logger TYPE REF TO zcl_logger.
DATA: ls_msg TYPE zcl_logger=>ty_message.

" Crea logger
CREATE OBJECT lo_logger
  EXPORTING
    iv_object = 'ZORDER'
    iv_subobject = 'PROCESSING'.

TRY.
    lo_processor->process_order( ).
    
    ls_msg-type = 'S'.
    ls_msg-message = 'Ordine processato correttamente'.
    lo_logger->add_message( ls_msg ).
    
  CATCH zcx_error INTO DATA(lx_error).
    ls_msg-type = 'E'.
    ls_msg-message = lx_error->get_text( ).
    ls_msg-details = lx_error->get_details( ).
    lo_logger->add_message( ls_msg ).
ENDTRY.

" Salva e mostra log
lo_logger->save_and_display( ).
```

---

## Log Levels

```abap
DATA: ls_message TYPE bal_s_msg.

" Informational
ls_message-msgtype = 'I'.
ls_message-msgtxt = 'Ordine creato con successo'.

" Warning
ls_message-msgtype = 'W'.
ls_message-msgtxt = 'Cliente ha credito insufficiente'.

" Error
ls_message-msgtype = 'E'.
ls_message-msgtxt = 'Errore: Cliente non trovato'.

" Success (Verde)
ls_message-msgtype = 'S'.
ls_message-msgtxt = 'Elaborazione completata'.

" Abort (Rosso, interrompe)
ls_message-msgtype = 'A'.
ls_message-msgtxt = 'Errore critico: Elaborazione interrotta'.
```

---

## Message Classes (SE91)

### Quando Usare Message Classes

Per messaggi **riutilizzabili e traducibili**:

```abap
" ✅ Messaggio con message class (traducibile, versionato)
MESSAGE e001(zorders) WITH 'ORD-123' 'Customer not found'.

" ❌ Messaggio hardcoded (non traducibile)
MESSAGE 'Errore: Cliente non trovato' TYPE 'E'.
```

### Esempio Message Class `ZORDERS`

```
MessageClass: ZORDERS

E 001: Cliente &1 non trovato. Contattare l'amministratore
I 002: Ordine &1 creato con successo
W 003: Ordine &1 ha credito insufficiente: &2
E 004: Errore durante salvataggio ordine &1
```

### Uso

```abap
MESSAGE e001(zorders) WITH lv_customer_id.
MESSAGE i002(zorders) WITH lv_order_id.
MESSAGE w003(zorders) WITH lv_order_id lv_credit_limit.
```

---

## Logging con Commenti Strutturati

### Template per Funzione Critica

```abap
METHOD process_order IMPORTING iv_order_id TYPE vbeln
                    RAISING zcx_order_error.
  
  " === INITIALIZATION ===
  DATA: lv_status TYPE string,
        lo_logger TYPE REF TO zcl_logger.
  
  CREATE OBJECT lo_logger
    EXPORTING iv_object = 'ZORDER'
              iv_subobject = 'PROCESS'.
  
  WRITE TO MESSAGE ID 'I' NUMBER 'Processing order' INTO lv_status.
  
  " === VALIDATION ===
  TRY.
      _validate_order( iv_order_id ).
    CATCH zcx_order_invalid INTO DATA(lx_invalid).
      lo_logger->add_message( TYPE='E' TEXT=lx_invalid->get_text( ) ).
      RAISE EXCEPTION lx_invalid.
  ENDTRY.
  
  " === PROCESSING ===
  TRY.
      _create_invoice( iv_order_id ).
      _update_inventory( iv_order_id ).
      _send_confirmation( iv_order_id ).
    CATCH cx_root INTO DATA(lx_error).
      lo_logger->add_message( TYPE='E' TEXT=lx_error->get_text( ) ).
      ROLLBACK WORK.
      RAISE EXCEPTION lx_error.
  ENDTRY.
  
  " === SUCCESS ===
  COMMIT WORK.
  lo_logger->add_message( TYPE='S' TEXT='Order processed successfully' ).
  lo_logger->save_and_display( ).
  
ENDMETHOD.
```

---

## Anti-Pattern (❌ Evita)

```abap
" ❌ Non loggare, solo messaggi
MESSAGE 'Errore durante processing' TYPE 'E'.

" ❌ Logging troppo verboso (every line)
lv_count = lv_count + 1.  " << Non loggare tutto!

" ❌ Messaggi hardcoded (non traducibili)
WRITE: / 'Errore critico'.

" ❌ Eccezioni non logggate
TRY.
    lo_obj->process( ).
  CATCH cx_root INTO DATA(lx).
    " Silenzio - BAD!
ENDTRY.
```

---

## Checklist Error Handling

- [ ] Ogni funzione critica ha TRY...CATCH
- [ ] Eccezioni specifiche catturate prima di generiche
- [ ] Errori sempre loggati (BAL o message)
- [ ] FINALLY per cleanup (file, connessioni)
- [ ] ROLLBACK dopo errore (transazioni)
- [ ] Utente informato (MESSAGE o LOG display)
- [ ] Log salvati in database (BAL_DB_SAVE)
- [ ] No exception ingoiate silenziosamente

---

**Ultimo aggiornamento**: Ottobre 2026
