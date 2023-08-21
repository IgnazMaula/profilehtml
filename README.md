Function UpdateJurnal(ByVal tReffNumber As String, ByVal tYear As String, ByVal tValue As Double, ByVal tPeriodNumber As Integer, ByVal tDate As Date, ByVal tUser As String, ByVal tDesc As String, _
Optional ByVal tCompany As String = "", Optional ByVal tJournalType As String = "DPPK", Optional ByVal tUnposted As Byte = 1, _
Optional ByVal tUnbalance As Byte = 0, Optional ByVal tCurr As String = "IDR", Optional ByVal tMD As String = "M", Optional ByVal tRate As Double = 1, Optional tExport As Byte = 0, _
Optional ByVal tBayarKepada As String = "", Optional ByVal tNilaiBayar As String = "0", Optional ByVal tBank As String = "", Optional ByVal tNoRekening As String = "") As Byte

    If tKredit < 0 Then
        tDebet = -tKredit
    ElseIf tDebet < 0 Then
        tKredit = -tDebet
    End If
    a = "insert into GLTRHDSUM(GLTRHD_ReffNumb,GLTRHD_Company,GLTRHD_FiscalYear,GLTRHD_BaseAmmt,GLTRHD_CurrAmmt,GLTRHD_JournType," & _
        "GLTRHD_Period,GLTRHD_Unposted,GLTRHD_Unbalanced,GLTRHD_EntryDate,GLTRHD_LastEntryDate,GLTRHD_EmpID,GLTRHD_LastEmpID," & _
        "GLTRHD_TransactionDate,GLTRHD_Currency,GLTRHD_MD,GLTRHD_ExchangeRate,GLTRHD_UserID,GLTRHD_Export,GLTRHD_Note," & _
        "GLTRHD_UserDef2, GLTRHD_UserDef3, GLTRHD_UserDef6, GLTRHD_UserDef7) values('" & _
        tReffNumber & _
        "','" & esc(tCompany) & _
        "','" & tYear & _
        "','" & Round(tValue, 0) & _
        "','" & Round(tValue, 0) & _
        "','" & tJournalType & _
        "','" & tPeriodNumber & _
        "','" & tUnposted & _
        "','" & tUnbalance & _
        "','" & Format(tDate, "yyyy/mm/dd") & _
        "','" & Format(tDate, "yyyy/mm/dd") & _
        "','" & tUser & _
        "','" & tUser & _
        "','" & Format(tDate, "yyyy/mm/dd") & _
        "','" & tCurr & _
        "','" & tMD & _
        "','" & tRate & _
        "','" & tUser & _
        "','" & tExport & _
        "','" & esc(tDesc) & _
        "','" & esc(tBayarKepada) & "','" & cNum(tNilaiBayar) & "','" & esc(tBank) & "','" & esc(tNoRekening) & "')"
    GDBConn.Execute a, b
    UpdateJurnal = b
    Exit Function
err:
    MsgBox err.Description
End Function
