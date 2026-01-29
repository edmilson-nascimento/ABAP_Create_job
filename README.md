
# ABAP Create job

Criação de JOB via código — cria e agenda JOBs em background (SM37) com exemplo prático.

![Static Badge](https://img.shields.io/badge/development-abap-blue)
![SAP](https://img.shields.io/badge/SAP-ABAP-0076D6)
![ABAP](https://img.shields.io/badge/Language-ABAP-006400)
![Eclipse%20ADT](https://img.shields.io/badge/Eclipse%20ADT-Recommended-6A1B9A)
![GitHub commit activity (branch)](https://img.shields.io/github/commit-activity/t/mourilo/ABAP_Create_job)
![Maintainer](https://img.shields.io/badge/murilo_borges-maintainer-lime)

## Sumário

- [Solução proposta](#solução-proposta)
- [Exemplo](#exemplo)
- [JDBG — Depuração de JOBs](#jdbg---depuração-de-jobs)
- [Pré-requisitos e autorizações](#pré-requisitos-e-autorizações)
- [Contribuindo](#contribuindo)

## Solução proposta
Aqui temos uma opção interessante para aquele processamento desagradável ou pesado demais que é mais interessante jogar pra um processamento em JOB/background e ter o LOG já na transação SM37, do que fazer todo um aparato de LOG para algo sem tanta relevância ou por não ter necessidade mesmo.
Mas fato é que aquele processo DEVE ser RODADO em JOB por algum motivo e para esse cenário temos uma receitinha SIMPLES prática e rápida.

## Exemplo
Pega a visão:

```abap

*-- Indica o tempo de a mais que será incluído para a data/hora de execução do JOB (isso dependende do processo que está executando, se não tiver necessidade joga fora).
CONSTANTS: c_diferenca_inicio TYPE sy-uzeit VALUE '000030'.

DATA: lv_jobid     TYPE btcjobcnt,
      lv_datestart TYPE sy-datum,
      lv_timestart TYPE sy-uzeit.

*-- Aqui é onde será criado o nome do JOB, deixa sua imaginação fluir.
DATA(lv_jobname) = conv btcjob( |DESC_JOB| ).

*--  // Cria o JOB //
CALL FUNCTION 'JOB_OPEN'
  EXPORTING
    jobname          = lv_jobname
  IMPORTING
    jobcount         = lv_jobid
  EXCEPTIONS
    cant_create_job  = 1
    invalid_job_data = 2
    jobname_missing  = 3
    OTHERS           = 4.
IF sy-subrc <> 0.
  RETURN.
ENDIF.

*--  // Agenda o programa de processamento no job criado com os parâmetros desejados //
SUBMIT nome_do_programa VIA JOB lv_jobname NUMBER lv_jobid
  WITH nome_parametro_programa = pametro_informado
   AND RETURN.
IF sy-subrc <> 0.
  RETURN.
ENDIF.

*--  // Calcula um novo "tempo" de início para o JOB //
CALL FUNCTION 'C14B_ADD_TIME'
  EXPORTING
    i_starttime = sy-uzeit
    i_startdate = sy-datum
    i_addtime   = c_diferenca_inicio
  IMPORTING
    e_endtime   = lv_timestart
    e_enddate   = lv_datestart.

IF sy-subrc <> 0.
  RETURN.
ENDIF.

*--  // Finaliza o JOB //
CALL FUNCTION 'JOB_CLOSE'
  EXPORTING
    jobcount             = lv_jobid
    jobname              = lv_jobname
    sdlstrtdt            = lv_datestart
    sdlstrttm            = lv_timestart
    strtimmed            = 'X'
  EXCEPTIONS
    cant_start_immediate = 1
    invalid_startdate    = 2
    jobname_missing      = 3
    job_close_failed     = 4
    job_nosteps          = 5
    job_notex            = 6
    lock_failed          = 7
    invalid_target       = 8
    OTHERS               = 9.
IF sy-subrc <> 0.
  RETURN.
ENDIF.
```

É isso, simples e prático.


## JDBG — Depuração de JOBs (SM37)

Caso precise depurar um JOB já executado (com sucesso ou erro), siga estes passos:

1. Na transação `SM37`, localize e selecione o JOB desejado.
2. Com a linha do JOB marcada, insira `JDBG` na área de comando e pressione Enter.
3. O sistema abrirá o fluxo do JOB; use F7 (Step Into) para entrar no programa e depurar.

**Dica prática:** Vá dando F7 até cair no programa do JOB e seja **FELIZ**.

4. Observações: pode ser necessário ter autorizações para visualizar/depurar JOBs (ex.: acesso à transação `SM37` e permissões relacionadas a jobs e debugging). Confirme com a equipe de Basis/Segurança antes de depurar em produção.

> Dica: "O mundo abrirá diante de seus olhos." Use com responsabilidade.

## Pré-requisitos e autorizações

- Acesso à transação `SM37` (p.ex.: via `S_TCODE`).
- Permissões para operar/visualizar JOBs (a autorização `S_BTCH_JOB` pode ser necessária).
- Permissões de depuração (consulte sua equipe de Basis/Segurança).
- Recomendação: prefira depurar em sistemas de desenvolvimento ou teste; em produção, obtenha aprovação e registre ações.

## Contribuindo

Se quiser contribuir, veja o arquivo `CONTRIBUTING.md` para orientações sobre issues, pull requests e padrões de commit.

