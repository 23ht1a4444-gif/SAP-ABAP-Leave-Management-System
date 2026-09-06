# Z549_LEAVE_REQUEST

## Description

`Z549_LEAVE_REQUEST` is an SAP ABAP report used to create and submit leave requests for employees.

The program collects the required leave details from the user and stores the leave request information in the `ZLEAVE_REQ0549` table.

## Purpose

- Allows employees to submit leave requests.
- Captures employee ID and leave details.
- Stores leave request information in the database.
- Maintains the initial status of the leave request.
- Supports the Leave Management System workflow.

## Usage

The program is used by employees to submit their leave requests. The submitted request is stored in `ZLEAVE_REQ0549` and can later be processed by the Manager Approval program.
