For i = 0 To Grid.Rows - 1
    tLunas = IIf(Grid.Columns("Lunas").Value <> 0 And Grid.Columns("Lunas").Value <> "", 1, 0)
    If Grid.Columns("Tanggal1").Value <> "" Then
        s = "select MFBNKCOM_KodeNoJurnal from MFBNKCOM where MFBNKCOM_BankID='" & Grid.Columns("Bank").Value & "' and MFBNKCOM_NoRekening='" & Grid.Columns("No Rekening").Value & "'"
        Set rs = Nothing
        rs.Open s, GDBConn, adOpenKeyset, adLockOptimistic
        tNoJurnal = GenerateNewNoJurnal("BM", rs!MFBNKCOM_KodeNoJurnal, Grid.Columns("Tanggal1").Value)
        
        s = "insert into Piutang(Type, Bilyet, Piutang, TanggalBunga, TanggalPiutang, PiutangBayar, Biaya, NoJurnal, Bank, NoRekening)" & _
" values('DEP','" & Grid.Columns("Bilyet").Value & "'," & cNum(Grid.Columns("Nilai Piutang").Value) & ",'" & Format(Grid.Columns("Tanggal").Value, "yyyy/mm/dd") & _
"','" & Format(Grid.Columns("Tanggal1").Value, "yyyy/mm/dd") & "','" & cNum(Grid.Columns("Piutang Bayar").Value) & "','" & cNum(Grid.Columns("Biaya").Value) & _
"','" & tNoJurnal & "','" & Grid.Columns("Bank").Value & "','" & Grid.Columns("No Rekening").Value & "')"
        GDBConn.Execute s
        tDate = Grid.Columns("Tanggal1").Value
        tYear = Year(tDate)
        tPeriodNo = Month(tDate)
        If UpdateJurnal(tNoJurnal, tYear, Grid.Columns("Piutang Bayar").Value, tPeriodNo, tDate, GUser, "Penerimaan Piutang Bilyet(" & Grid.Columns("Bilyet").Value & ")") = 0 Then GoTo err
        s = "select * from MI where MI_C='DEP'"
        Set rs = Nothing
        rs.Open s, GDBConn, adOpenKeyset, adLockOptimistic
        tAcc = rs!piutang
        tAccBiaya = rs!MI_Biaya
        tSubAccBiaya = rs!MI_SubAccBiaya
        s = "select SubAcc from TRD_D left join MFBNK on TRD_D_Bank=Bank_ID and TRD_D_Branch=Branch_ID where TRD_D_NoS='" & Grid.Columns("Bilyet").Value & "'"
        Set rs = Nothing
        rs.Open s, GDBConn, adOpenKeyset, adLockOptimistic
        tSubAcc = rs!subacc
        tBayar = IIf(Grid.Columns("Piutang Bayar").Value = "", 0, Grid.Columns("Piutang Bayar").Value)
        tBiaya = IIf(Grid.Columns("Biaya").Value = "", 0, Grid.Columns("Biaya").Value)
        If UpdateDetailJurnal(tNoJurnal, tLine, tAcc, tSubAcc, tDate, tKet, 0, tBayar - tBiaya, tYear, tPeriodNo) = 0 Then GoTo err
        tLine = tLine + 1
        s = "select * from MFBNKCOM where MFBNKCOM_BankID='" & Grid.Columns("Bank").Value & "' and MFBNKCOM_NoRekening='" & Grid.Columns("No Rekening").Value & "'"
        Set rs = Nothing
        rs.Open s, GDBConn, adOpenKeyset, adLockOptimistic
        tAcc = rs!MFBNKCOM_Account
        tSubAcc = rs!MFBNKCOM_SubAcc
        If UpdateDetailJurnal(tNoJurnal, tLine, tAcc, tSubAcc, tDate, tKet, tBayar, 0, tYear, tPeriodNo) = 0 Then GoTo err
        tLine = tLine + 1
        If UpdateDetailJurnal(tNoJurnal, tLine, tAccBiaya, tSubAccBiaya, tDate, tKet, tBiaya, 0, tYear, tPeriodNo) = 0 Then GoTo err
    End If
    s = "update TRD_REAL set Lunas='" & tLunas & "', Piutang=" & IIf(Grid.Columns("Piutang Bayar").Value > 0, 1, 0) & " where Bilyet='" & Grid.Columns("Bilyet").Text & "' and TanggalBayar='" & Format(Grid.Columns("Tanggal").Value, "yyyy/mm/dd") & "'"
    GDBConn.Execute s
    Grid.MoveNext
