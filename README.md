delete GLTRANSUM from GLTRANSUM inner join TRAKHIR on GLTRAN_ReffNumb=NoJurnal1 where TRAKHIR_Type='" & tipe & "' and TRAKHIR_Periode='" & Format(dtTanggal.Value, "yyyy/mm") & "'
