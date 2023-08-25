Function cNum(ByVal tNum As Variant, Optional ByVal tPrec As Byte = 30) As String
On Error GoTo err
    cNum = "0"
    cNum = Round(CDec(tNum), tPrec)
    If Left(Format(0, "###.00"), 1) = "," Then
        cNum = Replace(cNum, ",", ".", 1, 1)
    End If
    Exit Function
err:
    MsgBox err.Description
End Function
