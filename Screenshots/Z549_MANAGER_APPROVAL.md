*&---------------------------------------------------------------------*
*& Report Z549_MANAGER_APPROVAL
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT z549_manager_approval.

TABLES: zleave_req0549,
        zemployee_0549,
        sscrfields.

DATA: gs_req TYPE zleave_req0549,
      gs_emp TYPE zemployee_0549.

*---------------------------------------------------------------------*
* REQUEST ID
*---------------------------------------------------------------------*

PARAMETERS p_reqid TYPE zleave_req0549-reqst_id.

*---------------------------------------------------------------------*
* GET DETAILS BUTTON
*---------------------------------------------------------------------*

SELECTION-SCREEN PUSHBUTTON /1(15) btn_get USER-COMMAND getd.

*---------------------------------------------------------------------*
* EMPLOYEE DETAILS
*---------------------------------------------------------------------*

SELECTION-SCREEN COMMENT /1(25) text1.

PARAMETERS:
  p_empid  TYPE zemployee_0549-employee_id MODIF ID dsp,
  p_name   TYPE zemployee_0549-emp_name MODIF ID dsp,
  p_dept   TYPE zemployee_0549-e_department MODIF ID dsp,
  p_manid  TYPE zemployee_0549-manager_id MODIF ID dsp,
  p_status TYPE zemployee_0549-status MODIF ID dsp.

*---------------------------------------------------------------------*
* REQUEST STATUS
*---------------------------------------------------------------------*

PARAMETERS:
  p_reqsts TYPE char10 MODIF ID dsp.

*---------------------------------------------------------------------*
* LEAVE DETAILS
*---------------------------------------------------------------------*

PARAMETERS:
  p_ltype  TYPE zleave_req0549-leave_type MODIF ID dsp,
  p_from   TYPE zleave_req0549-from_date MODIF ID dsp,
  p_to     TYPE zleave_req0549-to_date MODIF ID dsp,
  p_days   TYPE zleave_req0549-no_of_days MODIF ID dsp,
  p_reason TYPE zleave_req0549-reason MODIF ID dsp.

*---------------------------------------------------------------------*
* ACTION
*---------------------------------------------------------------------*

SELECTION-SCREEN COMMENT /1(10) text2.

PARAMETERS:
  p_app RADIOBUTTON GROUP act DEFAULT 'X',
  p_rej RADIOBUTTON GROUP act.

*---------------------------------------------------------------------*
* INITIALIZATION
*---------------------------------------------------------------------*

INITIALIZATION.

  btn_get = 'GET DETAILS'.
  text1   = 'EMPLOYEE DETAILS'.
  text2   = 'ACTION'.

*---------------------------------------------------------------------*
* OUTPUT ONLY FIELDS
*---------------------------------------------------------------------*

AT SELECTION-SCREEN OUTPUT.

  LOOP AT SCREEN.

    IF screen-group1 = 'DSP'.

      screen-input = 0.
      MODIFY SCREEN.

    ENDIF.

  ENDLOOP.

*---------------------------------------------------------------------*
* GET DETAILS
*---------------------------------------------------------------------*

AT SELECTION-SCREEN.

  IF sscrfields-ucomm = 'GETD'.

    CLEAR: gs_req,
           gs_emp,
           p_empid,
           p_name,
           p_dept,
           p_manid,
           p_status,
           p_reqsts,
           p_ltype,
           p_from,
           p_to,
           p_days,
           p_reason.

    IF p_reqid IS INITIAL.
      MESSAGE 'Please enter Request ID' TYPE 'E'.
    ENDIF.

*---------------------------------------------------------------------*
* GET PENDING REQUEST
*---------------------------------------------------------------------*

    SELECT SINGLE *
      FROM zleave_req0549
      INTO gs_req
      WHERE reqst_id = p_reqid
        AND status   = 'P'.

    IF sy-subrc <> 0.
      MESSAGE 'Request ID not found or request is not pending' TYPE 'E'.
    ENDIF.

*---------------------------------------------------------------------*
* GET EMPLOYEE
*---------------------------------------------------------------------*

    SELECT SINGLE *
      FROM zemployee_0549
      INTO gs_emp
      WHERE employee_id = gs_req-employee_id.

    IF sy-subrc <> 0.
      MESSAGE 'Employee details not found' TYPE 'E'.
    ENDIF.

*---------------------------------------------------------------------*
* EMPLOYEE DETAILS
*---------------------------------------------------------------------*

    p_empid  = gs_emp-employee_id.
    p_name   = gs_emp-emp_name.
    p_dept   = gs_emp-e_department.
    p_manid  = gs_emp-manager_id.
    p_status = gs_emp-status.

*---------------------------------------------------------------------*
* REQUEST STATUS
*---------------------------------------------------------------------*

    IF gs_req-status = 'P'.
      p_reqsts = 'PENDING'.

    ELSEIF gs_req-status = 'APPROVED'.
      p_reqsts = 'APPROVED'.

    ELSEIF gs_req-status = 'REJECTED'.
      p_reqsts = 'REJECTED'.

    ENDIF.

*---------------------------------------------------------------------*
* LEAVE DETAILS
*---------------------------------------------------------------------*

    p_ltype  = gs_req-leave_type.
    p_from   = gs_req-from_date.
    p_to     = gs_req-to_date.
    p_days   = gs_req-no_of_days.
    p_reason = gs_req-reason.

    MESSAGE 'Request details loaded' TYPE 'S'.

  ENDIF.

*---------------------------------------------------------------------*
* START OF SELECTION
*---------------------------------------------------------------------*

START-OF-SELECTION.

  IF p_reqid IS INITIAL.
    MESSAGE 'Please enter Request ID' TYPE 'E'.
  ENDIF.

*---------------------------------------------------------------------*
* GET PENDING REQUEST
*---------------------------------------------------------------------*

  SELECT SINGLE *
    FROM zleave_req0549
    INTO gs_req
    WHERE reqst_id = p_reqid
      AND status   = 'P'.

  IF sy-subrc <> 0.
    MESSAGE 'Request is not pending or Request ID not found' TYPE 'E'.
  ENDIF.

*---------------------------------------------------------------------*
* GET EMPLOYEE
*---------------------------------------------------------------------*

  SELECT SINGLE *
    FROM zemployee_0549
    INTO gs_emp
    WHERE employee_id = gs_req-employee_id.

  IF sy-subrc <> 0.
    MESSAGE 'Employee details not found' TYPE 'E'.
  ENDIF.

*---------------------------------------------------------------------*
* APPROVE
*---------------------------------------------------------------------*

  IF p_app = 'X'.

    IF gs_emp-status = 'ACTIVE'.

      UPDATE zleave_req0549
        SET status      = 'APPROVED'
            approved_by = 'MANAGER'
        WHERE reqst_id = p_reqid
          AND status   = 'P'.

      IF sy-subrc = 0.

        COMMIT WORK.

        WRITE: / '=================================================='.
        WRITE: / '             LEAVE REQUEST RESULT'.
        WRITE: / '=================================================='.
        WRITE: /.
        WRITE: / 'Request ID      :', gs_req-reqst_id.
        WRITE: / 'Employee ID     :', gs_emp-employee_id.
        WRITE: / 'Employee Name   :', gs_emp-emp_name.
        WRITE: / 'Department      :', gs_emp-e_department.
        WRITE: / 'Manager ID      :', gs_emp-manager_id.
        WRITE: / 'Employee Status :', gs_emp-status.
        WRITE: / 'Leave Type      :', gs_req-leave_type.
        WRITE: / 'From Date       :', gs_req-from_date.
        WRITE: / 'To Date         :', gs_req-to_date.
        WRITE: / 'No. of Days     :', gs_req-no_of_days.
        WRITE: / 'Reason          :', gs_req-reason.
        WRITE: /.
        WRITE: / '--------------------------------------------------'.
        WRITE: / 'Leave Request   : APPROVED'.
        WRITE: / 'Approved By     : MANAGER'.
        WRITE: / '--------------------------------------------------'.

      ELSE.

        ROLLBACK WORK.
        MESSAGE 'Error while approving request' TYPE 'E'.

      ENDIF.

    ELSE.

*---------------------------------------------------------------------*
* EMPLOYEE INACTIVE - APPROVE CANNOT HAPPEN
*---------------------------------------------------------------------*

      UPDATE zleave_req0549
        SET status      = 'REJECTED'
            approved_by = 'MANAGER'
        WHERE reqst_id = p_reqid
          AND status   = 'P'.

      IF sy-subrc = 0.

        COMMIT WORK.

        WRITE: / '=================================================='.
        WRITE: / '             LEAVE REQUEST RESULT'.
        WRITE: / '=================================================='.
        WRITE: /.
        WRITE: / 'Request ID      :', gs_req-reqst_id.
        WRITE: / 'Employee ID     :', gs_emp-employee_id.
        WRITE: / 'Employee Name   :', gs_emp-emp_name.
        WRITE: / 'Department      :', gs_emp-e_department.
        WRITE: / 'Manager ID      :', gs_emp-manager_id.
        WRITE: / 'Employee Status :', gs_emp-status.
        WRITE: /.
        WRITE: / '--------------------------------------------------'.
        WRITE: / 'Employee is INACTIVE'.
        WRITE: / 'Leave Request   : REJECTED'.
        WRITE: / 'Rejected By     : MANAGER'.
        WRITE: / '--------------------------------------------------'.

      ELSE.

        ROLLBACK WORK.
        MESSAGE 'Error while rejecting request' TYPE 'E'.

      ENDIF.

    ENDIF.

*---------------------------------------------------------------------*
* REJECT
*---------------------------------------------------------------------*

  ELSEIF p_rej = 'X'.

    UPDATE zleave_req0549
      SET status      = 'REJECTED'
          approved_by = 'MANAGER'
      WHERE reqst_id = p_reqid
        AND status   = 'P'.

    IF sy-subrc = 0.

      COMMIT WORK.

      WRITE: / '=================================================='.
      WRITE: / '             LEAVE REQUEST RESULT'.
      WRITE: / '=================================================='.
      WRITE: /.
      WRITE: / 'Request ID      :', gs_req-reqst_id.
      WRITE: / 'Employee ID     :', gs_emp-employee_id.
      WRITE: / 'Employee Name   :', gs_emp-emp_name.
      WRITE: / 'Department      :', gs_emp-e_department.
      WRITE: / 'Manager ID      :', gs_emp-manager_id.
      WRITE: / 'Employee Status :', gs_emp-status.
      WRITE: / 'Leave Type      :', gs_req-leave_type.
      WRITE: / 'From Date       :', gs_req-from_date.
      WRITE: / 'To Date         :', gs_req-to_date.
      WRITE: / 'No. of Days     :', gs_req-no_of_days.
      WRITE: / 'Reason          :', gs_req-reason.
      WRITE: /.
      WRITE: / '--------------------------------------------------'.
      WRITE: / 'Leave Request   : REJECTED'.
      WRITE: / 'Rejected By     : MANAGER'.
      WRITE: / '--------------------------------------------------'.

    ELSE.

      ROLLBACK WORK.
      MESSAGE 'Error while rejecting request' TYPE 'E'.

    ENDIF.

  ENDIF.
