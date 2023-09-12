Public Function BuatNomorMemo(ByVal tanggal As Date, ByVal jenis As String) As String
On Error GoTo err
Dim str As String, rs As New ADODB.Recordset, Angka As Double
Dim bln As Double, Thn As Double

str = "select * from bulan where periode='" & Format(tanggal, "yyyy/mm") & "'"
Set rs = Nothing
rs.Open str, GDBConn, adOpenKeyset, adLockOptimistic
If rs.EOF Then
    rs.AddNew
    Angka = 0
Else
    Angka = rs!nomor
End If
rs!periode = Format(tanggal, "yyyy/mm")
rs!nomor = Angka + 1
bln = Angka + 1
rs.Update

str = "select * from tahun where tahun=" & Year(tanggal)
Set rs = Nothing
rs.Open str, GDBConn, adOpenKeyset, adLockOptimistic
If rs.EOF Then
    rs.AddNew
    Angka = 0
Else
    Angka = rs!nomor
End If
rs!tahun = Year(tanggal)
rs!nomor = Angka + 1
Thn = Angka + 1
rs.Update

BuatNomorMemo = bln & "/" & Thn & "/" & AngkaRomawi(Month(tanggal)) & "/" & Year(tanggal)

Exit Function
err:
BuatNomorMemo = ""
MsgBox err.Description, vbInformation
End Function
