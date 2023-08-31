Sub UpdateAverage(ByVal tdp As String, ByVal ttanggal As Date, ByVal tKode As String, ByVal tType As String, ByVal tUnit As Double, ByVal tNewPrice As Double)
    If tUnit < 0 Then
        s = "delete from AVG where DPCode='" & tdp & "' and Kode='" & tKode & "' and Type='" & tType & "' and -Unit=" & cNum(tUnit, 4)
        GDBConn.Execute s, b
        If b = 0 Then
            s = "update AVG set unit=unit+" & cNum(tUnit) & ", Average=(" & cNum(tUnit * tNewPrice) & "+Unit*Average)/(" & cNum(tUnit) & "+Unit) where DPCode='" & tdp & "' and Kode='" & tKode & "' and Type='" & tType & "'"
            GDBConn.Execute s, b
        End If
    Else
        s = "update AVG set unit=unit+" & cNum(tUnit) & ", AVGTotal=AVGTotal+" & cNum(tUnit * tNewPrice) & ", Average=(" & cNum(tUnit * tNewPrice) & "+Unit*Average)/(" & cNum(tUnit) & "+Unit) where DPCode='" & tdp & "' and Kode='" & tKode & "' and Type='" & tType & "'"
        's = "update AVG set unit=unit+" & cNum(tUnit) & ", AVGTotal=AVGTotal+" & cNum(tUnit * tNewPrice) & ", Average=" & cNum(HargaRata2vq) & " where DPCode='" & tdp & "' and Kode='" & tKode & "' and Type='" & tType & "'"
        
        GDBConn.Execute s, b
        If b = 0 Then
            s = "insert into AVG(DPCode, Tanggal, Kode, Unit, Average, Type) values('" & _
                tdp & _
                "','" & Format(ttanggal, "yyyy/mm/dd") & _
                "','" & tKode & _
                "','" & cNum(tUnit) & _
                "','" & cNum(tNewPrice) & _
                "','" & tType & "')"
            GDBConn.Execute s, b
        End If
    End If
End Sub
