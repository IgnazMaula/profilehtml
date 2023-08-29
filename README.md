Sub UpdateHarian(ByVal tKode As String, ByVal tType As String, ByVal tDate As Date, ByVal tNominal As Double, ByVal tKurs As Double, ByVal tJenisTransaksi As String, ByVal tdp As String, ByVal tPihak As String, ByVal tnama As String)
    On Error GoTo err
    Dim rs As New ADODB.Recordset, rs1 As New ADODB.Recordset
    Dim Saldo As Double, str As String
    Dim rsvq As New ADODB.Recordset
    If tPihak = "" Then
        str = "select MFPIHAK_Kode from MFPIHAK where MFPIHAK_Nama='" & tnama & "'"
        Set rs = Nothing
        rs.Open str, GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
        tPihak = rs!MFPIHAK_Kode
    End If
    str = "delete from AVGD where AVGD_DPCode='" & tdp & "' and AVGD_Kode='" & esc(tKode) & "' and AVGD_Tanggal>'" & Format(tDate, "yyyy/mm/dd") & "'"
    GDBConn.Execute str, b
    Set rs = Nothing
    str = "select max(AVGD_Tanggal) as My_Date from AVGD group by AVGD_DPCode,AVGD_Kode having AVGD_DPCode='" & tdp & "' and AVGD_Kode='" & esc(tKode) & "'"
    Set rs = Nothing
    rs.Open str, GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
    If rs.EOF Then
        Set rs = Nothing
        rs.Open "select top 0 * from AVGD", GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
        rs.AddNew
        rs!AVGD_Kode = tKode
        rs!avgd_type = tType
        rs!avgd_trans = tJenisTransaksi
        rs!AVGD_Tanggal = Format(tDate, "yyyy/mm/dd")
        rs!avgd_saldoawal = 0
        rs!avgd_unit = tNominal
        rs!AVGD_SaldoAkhir = tNominal
        rs!avgd_average = cNum(tKurs, 4)
        rs!AVGD_DPCode = tdp
        rs!avgd_pihak = tPihak
        rs!avgd_nama = tnama
        If tType = "OBL" Or tType = "HTM" Or tType = "SBP" Then
            rs!avgd_Total = CDbl(tNominal) * CDbl(tKurs) / 100
        Else
            rs!avgd_Total = CDbl(tNominal) * CDbl(tKurs)
        End If
        rs.Update
    Else
        If Format(rs!My_Date, "yyyy/mm/dd") = Format(tDate, "yyyy/mm/dd") Then
            str = "select * from AVGD where AVGD_Kode='" & esc(tKode) & "' and AVGD_DPCode='" & tdp & "' and AVGD_Tanggal='" & Format(rs!My_Date, "yyyy/mm/dd") & "'"
            Set rs1 = Nothing
            rs1.Open str, GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
        Else
            str = "select top 0 * from AVGD"
            Set rs1 = Nothing
            rs1.Open str, GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
            rs1.AddNew
        End If
        
        str = "select * from AVGD where AVGD_Kode='" & esc(tKode) & "' and AVGD_DPCode='" & tdp & "' and AVGD_Tanggal='" & Format(rs!My_Date, "yyyy/mm/dd") & "'"
        Set rs = Nothing
        rs.Open str, GDBConn, adOpenKeyset, adLockOptimistic, adCmdText
        'edited by vq nambah isnull
            'misal saldo akhir udah abis
            If tJenisTransaksi = "B" And rs!AVGD_SaldoAkhir = 0 Then
                rsvq.Open "select top 1 * from avg where kode='" & tKode & "'", GDBConn, 1, 3
                rs1!avgd_average = rsvq!Average
            End If
        Saldo = HargaRata2vq * IIf(IsNull(rs!AVGDSaldoAkhir), rs!AVGD_SaldoAkhir, rs!AVGDSaldoAkhir)

        rs1!AVGD_Kode = tKode
        rs1!avgd_type = tType
        rs1!avgd_trans = tJenisTransaksi
        rs1!AVGD_Tanggal = Format(tDate, "yyyy/mm/dd")
        'edited by vq tambah iif
        rs1!avgd_saldoawal = IIf(IsNull(rs!AVGDSaldoAkhir), rs!AVGD_SaldoAkhir, rs!AVGDSaldoAkhir)
        rs1!avgd_unit = IIf(tJenisTransaksi = "B", tNominal, -tNominal)
        rs1!AVGD_SaldoAkhir = IIf(tJenisTransaksi = "B", tNominal, -tNominal) + rs1!avgd_saldoawal
        If tType = "OBL" Or tType = "HTM" Or tType = "SBP" Then
        'edited by vq
            If tJenisTransaksi = "B" Then
                rs1!avgd_Total = (CDbl(tNominal) * CDbl(tKurs) / 100) + rs!avgdtotal
            Else
                '[2014.02.11] SBP FT
                'rs1!avgd_Total = cNum(rs!avgdtotal - (rs!avgdtotal * CDbl(tNominal) / CDbl(rs!AVGDSaldoAkhir)), 0)
                If IsNull(rs!AVGDSaldoAkhir) Then
                    rs1!avgd_Total = 0
                Else
                    rs1!avgd_Total = cNum(IIf(IsNull(rs!avgdtotal), 0, rs!avgdtotal) - (IIf(IsNull(rs!avgdtotal), 0, rs!avgdtotal) * CDbl(tNominal) / CDbl(rs!AVGDSaldoAkhir)), 0)
                End If
            End If
        Else
                Set rsvq = Nothing
                
                If tType = "SHM" Then
                    Set rsvq = Nothing
                     'str = "select sum(total) as total from trd_saham where Kodesaham='" & tKode & "' and tanggalbeli <= '" & Format(tDate, "yyyy/mm/dd") & "' and DP ='" & tdp & "' group by Kodesaham"
                    rsvq.Open "select sum(totalsaham) as total from trd_saham where Kodesaham='" & tKode & "' and tanggalsaham <= '" & Format(tDate, "yyyy/mm/dd") & "' and DP ='" & tdp & "' group by Kodesaham", GDBConn, 1, 3
                ElseIf tType = "RD" Then
                    Set rsvq = Nothing
                    rsvq.Open "select sum(totalRD) as total from trd_rd where KodeRD='" & tKode & "' and tanggalRD <= '" & Format(tDate, "yyyy/mm/dd") & "' and DP ='" & tdp & "' group by KodeRD", GDBConn, 1, 3
                End If

            If tJenisTransaksi = "B" Then
                If tType = "SHM" Or tType = "RD" Then
                    rs1!avgd_Total = rsvq!total
                Else
                    rs1!avgd_Total = (CDbl(tNominal) * CDbl(tKurs)) + IIf(IsNull(rs1!avgd_Total), 0, rs1!avgd_Total)
                End If
                
                '(CDbl(tNominal) * CDbl(tKurs)) + IIf(IsNull(rs1!avgd_Total), 0, rs1!avgd_Total)
            Else
                'edited by vq biar harga rata2 ndak berubah
                'rs1!avgd_total = cNum(IIf(IsNull(rs!avgdtotal), rs!avgd_total, rs!avgdtotal) - (IIf(IsNull(rs!avgdtotal), rs!avgd_total, rs!avgdtotal) * CDbl(tNominal) / CDbl(IIf(IsNull(rs!AVGDSaldoAkhir), rs!avgd_saldoakhir, rs!AVGDSaldoAkhir))), 0)
                If tType = "SHM" Or tType = "RD" Then
                    rs1!avgd_Total = rsvq!total
                Else
                    rs1!avgd_Total = rs!avgd_Total - (CDbl(tNominal) * CDbl(tKurs))
                End If
                'rs!avgd_Total - (CDbl(tNominal) * CDbl(tKurs))
                'rs1!avgd_total = rs!avgd_total
            End If
        End If
        If rs1!AVGD_SaldoAkhir = 0 Then
            'tambah else if untuk pembelian dan penjualan
            If tJenisTransaksi = "B" Then
            Set rsvq = Nothing
                rsvq.Open "select top 1 * from avg where kode='" & tKode & "'", GDBConn, 1, 3
                If rsvq.EOF Then
                    rs1!avgd_average = 0
                Else
                    rs1!avgd_average = rsvq!Average
                End If
            Else
                rs1!avgd_average = 0
            End If
        Else
            'edited by vq klo sell harga rata2 gax
            
            
            If tJenisTransaksi <> "B" Then
                rs1!avgd_average = HargaRata2vq
            Else
                rs1!avgd_average = cNum(rs1!avgd_Total / rs1!AVGD_SaldoAkhir, 4)
            End If
            rs1!avgd_Total = rs1!avgd_average * rs1!AVGD_SaldoAkhir
        End If
        rs1!AVGD_DPCode = tdp
        rs1!avgd_pihak = tPihak
        rs1!avgd_nama = tnama
        rs1.Update
        
    End If
    Set rs = Nothing
Exit Sub
err:
    MsgBox err.Description, vbInformation
End Sub
