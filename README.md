Function UpdateDetailJurnal(ByVal tReffNumber As String, ByVal tLine As Integer, ByVal tAcc As String, ByVal tSubAcc As String, _
ByVal tDate As Date, ByVal tDesc As String, ByVal tDebet As Double, ByVal tKredit As Double, ByVal tYear As String, ByVal tPeriodNumber As String, _
Optional ByVal tJenisJurnal As String = "DPPK", Optional ByVal tEntity As String = "", Optional ByVal tProject As String = "", Optional ByVal tCurr As String = "IDR", Optional ByVal tRate As Double = 1, Optional ByVal tModule As String = "", Optional ByVal tUnposted As Byte = 1) As Byte
On Error GoTo err
    If tKredit < 0 Then
        tDebet = -tKredit
        tKredit = 0
    ElseIf tDebet < 0 Then
        tKredit = -tDebet
        tDebet = 0
    End If
    If tDebet + tKredit = 0 Then
        UpdateDetailJurnal = 1
        Exit Function
    End If
    
    tDesc = ConvertAcn(tAcc, "A") & " - " & ConvertAcn(tSubAcc, "S")
    
    a = "insert into GLTRANSUM(GLTRAN_Company,GLTRAN_ReffNumb,GLTRAN_LineNo,GLTRAN_AccountCode,GLTRAN_JenisJurnal,GLTRAN_SubAccount," & _
        "GLTRAN_TransDate,GLTRAN_Entity,GLTRAN_Description,GLTRAN_Project,GLTRAN_CurrencyCode,GLTRAN_DebitAmount,GLTRAN_CreditAmount," & _
        "GLTRAN_BaseDebetAmount,GLTRAN_BaseCreditAmount,GLTRAN_CuEffDate,GLTRAN_CuRate,GLTRAN_FiscalYear,GLTRAN_Module,GLTRAN_PeriodEntry,GLTRAN_PeriodPost,GLTRAN_Unposted) values('" & _
        "','" & tReffNumber & _
        "','" & tLine & _
        "','" & tAcc & _
        "','" & tJenisJurnal & _
        "','" & tSubAcc & _
        "','" & Format(tDate, "yyyy/mm/dd") & _
        "','" & esc(tEntity) & _
        "','" & esc(tDesc) & _
        "','" & esc(tProject) & _
        "','" & tCurr & _
        "','" & cNum(tDebet) & _
        "','" & cNum(tKredit) & _
        "','" & cNum(tDebet) & _
        "','" & cNum(tKredit) & _
        "','" & Format(tDate, "yyyy/mm/dd") & _
        "','" & tRate & _
        "','" & tYear & _
        "','" & tModule & _
        "','" & tPeriodNumber & _
        "','" & tPeriodNumber & _
        "','" & tUnposted & "')"
    GDBConn.Execute a, b
    UpdateDetailJurnal = b
    Exit Function
err:
    MsgBox err.Description, vbInformation
End Function
