# ABAP_Create_job
Criação de JOB via código

Aqui temos uma opção interessante para aquele processamento desagradável ou pesado demais que é mais interessante jogar pra um processamento em JOB/background e ter o LOG já na transação SM37, do que fazer todo um aparato de LOG para algo sem tanta relevância ou por não ter necessidade mesmo.
Mas fato é que aquele processo DEVE ser RODADO em JOB por algum motivo e para esse cenário temos uma receitinha SIMPLES prática e rápida....

Pega a visão:

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


É isso, simples e prático...


### DICA IMPORTANTE ###
Caso precise debugar um JOB que já aconteceu (com erro ou sucesso) basta selecionar a linha desse JOB na SM37 (que queira debugar) e após marcar a linha inserir no local da transação a sigla "JDBG" (sem aspas) e dar ENTER.
O mundo abrirá diante de seus olhos.
Vá dando F7 até cair no programa do JOB e seja FELIZ...
