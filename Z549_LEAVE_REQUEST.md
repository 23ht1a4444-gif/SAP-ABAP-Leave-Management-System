*&---------------------------------------------------------------------*
*& Report Z549_LEAVE_REQUEST
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT z549_leave_request.

SELECTION-SCREEN COMMENT /35(30) text-001.

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-002.
PARAMETERS p_empid TYPE zemployee_0549-employee_id.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-003.
PARAMETERS p_ltype TYPE zemp_leave0549-leave_type.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-004.
PARAMETERS p_from TYPE zleave_req0549-from_date.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-005.
PARAMETERS p_to TYPE zleave_req0549-to_date.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-006.
PARAMETERS p_reason TYPE char100.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN BEGIN OF LINE.
SELECTION-SCREEN COMMENT 1(15) text-007.
PARAMETERS p_days TYPE i.
SELECTION-SCREEN END OF LINE.

SELECTION-SCREEN PUSHBUTTON /35(20) text-008 USER-COMMAND SUBM.

SELECTION-SCREEN END OF BLOCK b1.


DATA:
  lv_days     TYPE i,
  lv_reqst_id TYPE zleave_req0549-reqst_id,
  ls_req      TYPE zleave_req0549.


*---------------------------------------------------------------------*
* CALCULATE DAYS AND MAKE DAYS OUTPUT ONLY
*---------------------------------------------------------------------*

AT SELECTION-SCREEN OUTPUT.

  IF p_from IS NOT INITIAL
     AND p_to IS NOT INITIAL.

    IF p_to >= p_from.

      lv_days = p_to - p_from + 1.
      p_days = lv_days.

    ELSE.

      CLEAR p_days.

    ENDIF.

  ENDIF.


  LOOP AT SCREEN.

    IF screen-name = 'P_DAYS'.

      screen-input = 0.
      MODIFY SCREEN.

    ENDIF.

  ENDLOOP.


*---------------------------------------------------------------------*
* SUBMIT REQUEST
*---------------------------------------------------------------------*

AT SELECTION-SCREEN.

  IF sy-ucomm = 'SUBM'.

*---------------------------------------------------------------------*
* VALIDATIONS
*---------------------------------------------------------------------*

    IF p_empid IS INITIAL.
      MESSAGE 'Please enter Employee ID' TYPE 'E'.
    ENDIF.

    IF p_ltype IS INITIAL.
      MESSAGE 'Please enter Leave Type' TYPE 'E'.
    ENDIF.

    IF p_from IS INITIAL.
      MESSAGE 'Please enter From Date' TYPE 'E'.
    ENDIF.

    IF p_to IS INITIAL.
      MESSAGE 'Please enter To Date' TYPE 'E'.
    ENDIF.

    IF p_to < p_from.
      MESSAGE 'To Date cannot be earlier than From Date' TYPE 'E'.
    ENDIF.

    IF p_reason IS INITIAL.
      MESSAGE 'Please enter Reason' TYPE 'E'.
    ENDIF.


*---------------------------------------------------------------------*
* CALCULATE DAYS
*---------------------------------------------------------------------*

    lv_days = p_to - p_from + 1.


*---------------------------------------------------------------------*
* PREPARE LEAVE REQUEST
*---------------------------------------------------------------------*

    CLEAR ls_req.

    ls_req-employee_id = p_empid.
    ls_req-leave_type  = p_ltype.
    ls_req-from_date   = p_from.
    ls_req-to_date     = p_to.
    ls_req-no_of_days  = lv_days.
    ls_req-reason      = p_reason.

    "P = Pending
    ls_req-status      = 'P'.

    CLEAR ls_req-approved_by.

    ls_req-created_on  = sy-datum.


*---------------------------------------------------------------------*
* GENERATE REQUEST ID
*---------------------------------------------------------------------*

    SELECT MAX( reqst_id )
      INTO @lv_reqst_id
      FROM zleave_req0549.

    IF lv_reqst_id IS INITIAL.

      lv_reqst_id = 1.

    ELSE.

      lv_reqst_id = lv_reqst_id + 1.

    ENDIF.


    ls_req-reqst_id = lv_reqst_id.


*---------------------------------------------------------------------*
* INSERT REQUEST
*---------------------------------------------------------------------*

    INSERT zleave_req0549 FROM ls_req.

    IF sy-subrc = 0.

      COMMIT WORK.

      MESSAGE 'Leave request submitted successfully and is PENDING'
              TYPE 'S'.

    ELSE.

      ROLLBACK WORK.

      MESSAGE 'Error while submitting leave request'
              TYPE 'E'.

    ENDIF.

  ENDIF.
