 else if (request.JenisInvestasi == "Surat Berharga Pemerintah")
                {
                    if (request.TanggalBayar < DateTime.Parse(tanggalProses))
                    {
                        response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tanggal Salah.");
                        return response;
                    }

                    if (request.JenisTransaksi == "B")
                    {
                        var queryA = from mfbrk in _context.Mfbrks
                                     join trd_oh in _context.TrdOhSbps on mfbrk.MfbrkBroker equals trd_oh.Sekuritas
                                     into trd_ohGroup
                                     from trdOhLeft in trd_ohGroup.DefaultIfEmpty()
                                     join mobl in _context.Mobls on trdOhLeft.TrdOhKode equals mobl.MoblCode
                                     into moblGroup
                                     from moblLeft in moblGroup.DefaultIfEmpty()
                                     join mi in _context.Mis on trdOhLeft.TrdOhInvCode equals mi.MiC
                                     into miGroup
                                     from miLeft in miGroup.DefaultIfEmpty()
                                     where trdOhLeft.TrdOhDp == request.DpCode &&
                                           trdOhLeft.TrdOhNo == noSurat &&
                                           moblLeft.MoblType == "SBP"
                                     select new
                                     {
                                         MFBRK = mfbrk,
                                         TRD_OH = trdOhLeft,
                                         MOBL = moblLeft,
                                         MI = miLeft
                                     };

                        var resultA = queryA.FirstOrDefault();


                        if (resultA != null)
                        {
                            if (request.JenisTransaksi == "B")
                            {
                                HutPiut = resultA.MI.MiHutInves;
                                SubAcn = resultA.MI.MiSubAccount;
                            }
                            else
                            {
                                HutPiut = resultA.MI.MiPiutInves;
                                SubAcn = resultA.MI.MiSubAccount;
                            }
                        }
                        else
                        {
                            response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tidak dapat Disimpan karena Account & Sub Account Investasi belum ada");
                            return response;
                        }

                        Ket = resultA.TRD_OH.Ket;
                        KetHutPiut = _sharedFunction.ConvertAcn(HutPiut, "A") + " " + _sharedFunction.ConvertAcn(SubAcn, "S");
                    }
                    else
                    {
                        var queryA = from mfbrk in _context.Mfbrks
                                     join trd_oh in _context.TrdOhSbpSells on mfbrk.MfbrkBroker equals trd_oh.Sekuritas
                                     into trd_ohGroup
                                     from trdOhLeft in trd_ohGroup.DefaultIfEmpty()
                                     join mobl in _context.Mobls on trdOhLeft.TrdOhKode equals mobl.MoblCode
                                     into moblGroup
                                     from moblLeft in moblGroup.DefaultIfEmpty()
                                     join mi in _context.Mis on trdOhLeft.TrdOhInvCode equals mi.MiC
                                     into miGroup
                                     from miLeft in miGroup.DefaultIfEmpty()
                                     where trdOhLeft.TrdOhDp == request.DpCode &&
                                           trdOhLeft.TrdOhNo == noSurat &&
                                           moblLeft.MoblType == "SBP"
                                     select new
                                     {
                                         MFBRK = mfbrk,
                                         TRD_OH = trdOhLeft,
                                         MOBL = moblLeft,
                                         MI = miLeft
                                     };

                        var resultA = queryA.FirstOrDefault();

                        if (resultA != null)
                        {
                            if (request.JenisTransaksi == "B")
                            {
                                HutPiut = resultA.MI.MiHutInves;
                                SubAcn = resultA.MI.MiSubAccount;
                            }
                            else
                            {
                                HutPiut = resultA.MI.MiPiutInves;
                                SubAcn = resultA.MI.MiSubAccount;
                            }
                        }
                        else
                        {
                            response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tidak dapat Disimpan karena Account & Sub Account Investasi belum ada");
                            return response;
                        }

                        Ket = resultA.TRD_OH.Ket;
                        KetHutPiut = _sharedFunction.ConvertAcn(HutPiut, "A") + " " + _sharedFunction.ConvertAcn(SubAcn, "S");
                    }

                    //generate jurnal per BANK KELUAR
                    if (request.JenisTransaksi == "B")
                    {
                        var queryB = from trdepoOh in _context.TrdepoSbps
                                     join mfbnkcom in _context.Mfbnkcoms on new { Bank = trdepoOh.Bank, Rekening = trdepoOh.Cabang } equals new { Bank = mfbnkcom.MfbnkcomBankId, Rekening = mfbnkcom.MfbnkcomNoRekening } into mfbnkcomGroup
                                     from mfbnkcomLeft in mfbnkcomGroup.DefaultIfEmpty()
                                     where trdepoOh.Dpcode == request.DpCode &&
                                           trdepoOh.NoSurat == noSurat &&
                                           trdepoOh.Type == "G"
                                     select new
                                     {
                                         TRDEPO_OH = trdepoOh,
                                         MFBNKCOM = mfbnkcomLeft
                                     };

                        var resultB = queryB.ToList();

                        foreach (var item in resultB)
                        {
                            KetBank = _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomAccount, "A") + " " + _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomSubAcc, "S");
                            reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);

                            //Validasi sudah digunakan atau belum No. Ref Jurnalnya
                            var queryC = $"Select GLTRHD_ReffNumb FROM GLTRHDSUM WHERE GLTRHD_ReffNumb = '{reff}'";
                            var resultC = _context.FromSqlRaw<Gltransum>(queryC).ToList();

                            foreach (var gl in resultC)
                            {
                                reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);
                            }

                            AllReff = AllReff + ",'" + reff + "'";

                            //bank - Kredit
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 1, item.MFBNKCOM.MfbnkcomAccount, item.MFBNKCOM.MfbnkcomSubAcc, request.TanggalBayar, KetBank, 0, item.TRDEPO_OH.Jumlah.Value, yr, Period.Value, "DPPK", "", "Arus Kas", "IDR", 1, "DPPK", 1);

                            //Hutang Investasi - Debet
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 2, HutPiut, SubAcn, request.TanggalBayar, KetHutPiut, item.TRDEPO_OH.Jumlah.Value, 0, yr, Period.Value, "DPPK", "", "DPPK", "IDR", 1, "DPPK", 1);

                            //generate GLTRHDSUM
                            await _sharedFunction.UpdateJurnal(cancellationToken, reff, yr, item.TRDEPO_OH.Jumlah.Value, Period.Value, request.TanggalBayar, "ADM", Ket, "", "DPPK", 1, 0, "IDR", "M", 1, 0);

                            tBank = item.MFBNKCOM.MfbnkcomBankId + " - " + item.MFBNKCOM.MfbnkcomNoRekening;

                            if (request.JenisTransaksi == "S")
                            {
                                var queryD = $"update GLTRHDSUM set GLTRHD_UserDef3='{_sharedFunction.CNum(-item.TRDEPO_OH.Jumlah.Value)}', GLTRHD_UserDef9=1, GLTRHD_UserDef5='', GLTRHD_UserDef6='{item.MFBNKCOM.MfbnkcomBankId}', GLTRHD_UserDef7='{item.MFBNKCOM.MfbnkcomNoRekening}', GLTRHD_UserDef2='{BayarKe}' where GLTRHD_ReffNumb='{reff}'";
                                var resultD = _context.ExecuteSqlRaw(queryD);
                            }
                            else
                            {
                                var queryD = $"update GLTRHDSUM set GLTRHD_UserDef3='{_sharedFunction.CNum(item.TRDEPO_OH.Jumlah.Value)}', GLTRHD_UserDef9=1, GLTRHD_UserDef5='', GLTRHD_UserDef6='{item.MFBNKCOM.MfbnkcomBankId}', GLTRHD_UserDef7='{item.MFBNKCOM.MfbnkcomNoRekening}', GLTRHD_UserDef2='{BayarKe}' where GLTRHD_ReffNumb='{reff}'";
                                var resultD = _context.ExecuteSqlRaw(queryD);
                            }

                            total += item.TRDEPO_OH.Jumlah.Value;
                        }

                    }
                    else
                    {
                        var queryB = from trdepoOh in _context.TrdepoSbpSells
                                     join mfbnkcom in _context.Mfbnkcoms on new { Bank = trdepoOh.Bank, Rekening = trdepoOh.Cabang } equals new { Bank = mfbnkcom.MfbnkcomBankId, Rekening = mfbnkcom.MfbnkcomNoRekening } into mfbnkcomGroup
                                     from mfbnkcomLeft in mfbnkcomGroup.DefaultIfEmpty()
                                     where trdepoOh.Dpcode == request.DpCode &&
                                           trdepoOh.NoSurat == noSurat &&
                                           trdepoOh.Type == "G"
                                     select new
                                     {
                                         TRDEPO_OH = trdepoOh,
                                         MFBNKCOM = mfbnkcomLeft
                                     };

                        var resultB = queryB.ToList();

                        foreach (var item in resultB)
                        {
                            KetBank = _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomAccount, "A") + " " + _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomSubAcc, "S");
                            reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);

                            //Validasi sudah digunakan atau belum No. Ref Jurnalnya
                            var queryC = $"Select GLTRHD_ReffNumb FROM GLTRHDSUM WHERE GLTRHD_ReffNumb = '{reff}'";
                            var resultC = _context.FromSqlRaw<Gltransum>(queryC).ToList();

                            foreach (var gl in resultC)
                            {
                                reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);
                            }

                            AllReff = AllReff + ",'" + reff + "'";

                            //bank - Kredit
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 1, item.MFBNKCOM.MfbnkcomAccount, item.MFBNKCOM.MfbnkcomSubAcc, request.TanggalBayar, KetBank, 0, item.TRDEPO_OH.Jumlah.Value, yr, Period.Value, "DPPK", "", "Arus Kas", "IDR", 1, "DPPK", 1);

                            //Hutang Investasi - Debet
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 2, HutPiut, SubAcn, request.TanggalBayar, KetHutPiut, item.TRDEPO_OH.Jumlah.Value, 0, yr, Period.Value, "DPPK", "", "DPPK", "IDR", 1, "DPPK", 1);

                            //generate GLTRHDSUM
                            await _sharedFunction.UpdateJurnal(cancellationToken, reff, yr, item.TRDEPO_OH.Jumlah.Value, Period.Value, request.TanggalBayar, "ADM", Ket, "", "DPPK", 1, 0, "IDR", "M", 1, 0);

                            tBank = item.MFBNKCOM.MfbnkcomBankId + " - " + item.MFBNKCOM.MfbnkcomNoRekening;

                            if (request.JenisTransaksi == "S")
                            {
                                var queryD = $"update GLTRHDSUM set GLTRHD_UserDef3='{_sharedFunction.CNum(-item.TRDEPO_OH.Jumlah.Value)}', GLTRHD_UserDef9=1, GLTRHD_UserDef5='', GLTRHD_UserDef6='{item.MFBNKCOM.MfbnkcomBankId}', GLTRHD_UserDef7='{item.MFBNKCOM.MfbnkcomNoRekening}', GLTRHD_UserDef2='{BayarKe}' where GLTRHD_ReffNumb='{reff}'";
                                var resultD = _context.ExecuteSqlRaw(queryD);
                            }
                            else
                            {
                                var queryD = $"update GLTRHDSUM set GLTRHD_UserDef3='{_sharedFunction.CNum(item.TRDEPO_OH.Jumlah.Value)}', GLTRHD_UserDef9=1, GLTRHD_UserDef5='', GLTRHD_UserDef6='{item.MFBNKCOM.MfbnkcomBankId}', GLTRHD_UserDef7='{item.MFBNKCOM.MfbnkcomNoRekening}', GLTRHD_UserDef2='{BayarKe}' where GLTRHD_ReffNumb='{reff}'";
                                var resultD = _context.ExecuteSqlRaw(queryD);
                            }

                            total += item.TRDEPO_OH.Jumlah.Value;
                        }
                    }

                    //Update Flag Bayar
                    if (request.JenisTransaksi == "B")
                    {
                        var queryE = $"update TRD_OH_SBP set Bayar=1 where TRD_OH_DP='{request.DpCode}' and TRD_OH_No='{noSurat}'";
                        var resultE = _context.ExecuteSqlRaw(queryE);

                        var TrdOH = _context.TrdOhSbps
                        .Where(o => o.TrdOhDp == "DP Smart" && o.TrdOhNo == "noSurat")
                        .GroupBy(o => 1)
                        .Select(g => new
                        {
                            DiskontoAmt = g.Sum(o => o.Diskonto),
                            PPhAmt = g.Sum(o => o.Pph),
                            TotAmt = g.Sum(o => o.Nominal * o.Kurs / 100)
                        })
                        .FirstOrDefault();
                        if (TrdOH != null)
                        {
                            DisAmt = TrdOH.DiskontoAmt.Value;
                            PjkAmt = TrdOH.PPhAmt.Value;
                            TransAmt = TrdOH.TotAmt.Value;
                        }

                        //update TRLAK
                        var queryF = $"select * from MSRPT where MSRPT_Modul ='BPI'";
                        var resultF = _context.FromSqlRaw<Msrpt>(queryF).FirstOrDefault();

                        if (resultF != null)
                        {
                            var queryG = $"select * from TRLAK where TRLAK_ReportCode = '{resultF.MsrptKode}' and TRLAK_BarisCode = '{resultF.MsrptBarisCode}' and TRLAK_Period = '{MPer}'";
                            var resultG = _context.FromSqlRaw<Trlak>(queryG).FirstOrDefault();

                            if (resultG != null)
                            {
                                var newTRLAK = new Trlak
                                {
                                    TrlakReportCode = resultF.MsrptKode,
                                    TrlakBarisCode = resultF.MsrptBarisCode,
                                    TrlakPeriod = MPer
                                };

                                newTRLAK.TrlakAmount = (resultG.TrlakAmount == null ? 0 : resultG.TrlakAmount) + TransAmt;

                                _context.Trlaks.Add(newTRLAK);
                                await _context.SaveChangesAsync(cancellationToken);
                            }
                        }

                        //Kalau ada DISKONTO
                        if (DisAmt > 0)
                        {
                            var queryH = $"select * from MSRPT where MSRPT_Modul ='PBR'";
                            var resultH = _context.FromSqlRaw<Msrpt>(queryH).FirstOrDefault();

                            if (resultH != null)
                            {
                                var queryI = $"select * from TRLAK where TRLAK_ReportCode = '{resultH.MsrptKode}' and TRLAK_BarisCode = '{resultH.MsrptBarisCode}' and TRLAK_Period = '{MPer}'";
                                var resultI = _context.FromSqlRaw<Trlak>(queryI).FirstOrDefault();

                                if (resultI != null)
                                {
                                    var newTRLAK = new Trlak
                                    {
                                        TrlakReportCode = resultH.MsrptKode,
                                        TrlakBarisCode = resultH.MsrptBarisCode,
                                        TrlakPeriod = MPer
                                    };

                                    newTRLAK.TrlakAmount = (resultI.TrlakAmount == null ? 0 : resultI.TrlakAmount) - DisAmt;

                                    _context.Trlaks.Add(newTRLAK);
                                    await _context.SaveChangesAsync(cancellationToken);
                                }
                            }
                        }

                        //Kalau ada PAJAK
                        if (DisAmt > 0)
                        {
                            var queryH = $"select * from MSRPT where MSRPT_Modul ='PPH'";
                            var resultH = _context.FromSqlRaw<Msrpt>(queryH).FirstOrDefault();

                            if (resultH != null)
                            {
                                var queryI = $"select * from TRLAK where TRLAK_ReportCode = '{resultH.MsrptKode}' and TRLAK_BarisCode = '{resultH.MsrptBarisCode}' and TRLAK_Period = '{MPer}'";
                                var resultI = _context.FromSqlRaw<Trlak>(queryI).FirstOrDefault();

                                if (resultI != null)
                                {
                                    var newTRLAK = new Trlak
                                    {
                                        TrlakReportCode = resultH.MsrptKode,
                                        TrlakBarisCode = resultH.MsrptBarisCode,
                                        TrlakPeriod = MPer
                                    };

                                    newTRLAK.TrlakAmount = (resultI.TrlakAmount == null ? 0 : resultI.TrlakAmount) - PjkAmt;

                                    _context.Trlaks.Add(newTRLAK);
                                    await _context.SaveChangesAsync(cancellationToken);
                                }
                            }
                        }

                    }
                    else
                    {
                        var queryE = $"update TRD_OH_SBP_SELL set Bayar=1 where TRD_OH_DP='{request.DpCode}' and TRD_OH_No='{noSurat}'";
                        var resultE = _context.ExecuteSqlRaw(queryE);

                        //update TRLAK
                        var queryF = $"select * from MSRPT where MSRPT_Modul ='PPI'";
                        var resultF = _context.FromSqlRaw<Msrpt>(queryF).FirstOrDefault();

                        if (resultF != null)
                        {
                            var queryG = $"select * from TRLAK where TRLAK_ReportCode = '{resultF.MsrptKode}' and TRLAK_BarisCode = '{resultF.MsrptBarisCode}' and TRLAK_Period = '{MPer}'";
                            var resultG = _context.FromSqlRaw<Trlak>(queryG).FirstOrDefault();

                            if (resultG != null)
                            {
                                var newTRLAK = new Trlak
                                {
                                    TrlakReportCode = resultF.MsrptKode,
                                    TrlakBarisCode = resultF.MsrptBarisCode,
                                    TrlakPeriod = MPer
                                };

                                newTRLAK.TrlakAmount = (resultG.TrlakAmount == null ? 0 : resultG.TrlakAmount) + total;

                                _context.Trlaks.Add(newTRLAK);
                                await _context.SaveChangesAsync(cancellationToken);
                            }
                        }
                    }
                }
