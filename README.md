else if (request.JenisInvestasi == "Reksadana")
                {
                    if (request.TanggalBayar < DateTime.Parse(tanggalProses))
                    {
                        response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tanggal Salah.");
                        return response;
                    }

                    if (request.JenisTransaksi == "B")
                    {
                        var queryA = from mfbrk in _context.Mfbrks
                                     join trdRd in _context.TrdRds on mfbrk.MfbrkBroker equals trdRd.Broker
                                     into trdRdGroup
                                     from trdRd in trdRdGroup.DefaultIfEmpty()
                                     join mshm in _context.Mshms on trdRd.KodeRd equals mshm.MshmC
                                     into mshmGroup
                                     from mshm in mshmGroup.DefaultIfEmpty()
                                     join mi in _context.Mis on trdRd.InvestCode equals mi.MiC
                                     into miGroup
                                     from mi in miGroup.DefaultIfEmpty()
                                     where trdRd.NoKonfirmasiBroker == noSurat && trdRd.Total > 0
                                     select new
                                     {
                                         MFBRK = mfbrk,
                                         TRD_RD = trdRd,
                                         MSHM = mshm,
                                         MI = mi
                                     };

                        var resultA = queryA.FirstOrDefault();

                        if (resultA != null)
                        {
                            HutPiut = resultA.MI.MiHutInves;
                            SubAcn = resultA.MI.MiSubAccount;
                            BayarKe = resultA.MFBRK.MfbrkBroker;
                        }
                        else
                        {
                            response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tidak dapat Disimpan karena Account & Sub Account Investasi belum ada");
                            return response;
                        }

                        Ket = resultA.TRD_RD.Ket;
                    }
                    else
                    {
                        var queryA = from mfbrk in _context.Mfbrks
                                     join trdRd in _context.TrdRds on mfbrk.MfbrkBroker equals trdRd.Broker
                                     into trdRdGroup
                                     from trdRd in trdRdGroup.DefaultIfEmpty()
                                     join mshm in _context.Mshms on trdRd.KodeRd equals mshm.MshmC
                                     into mshmGroup
                                     from mshm in mshmGroup.DefaultIfEmpty()
                                     join mi in _context.Mis on trdRd.InvestCode equals mi.MiC
                                     into miGroup
                                     from mi in miGroup.DefaultIfEmpty()
                                     where trdRd.NoKonfirmasiBroker == noSurat && trdRd.Total < 0
                                     select new
                                     {
                                         MFBRK = mfbrk,
                                         TRD_RD = trdRd,
                                         MSHM = mshm,
                                         MI = mi
                                     };

                        var resultA = queryA.FirstOrDefault();

                        if (resultA != null)
                        {
                            HutPiut = resultA.MI.MiPiutInves;
                            SubAcn = resultA.MI.MiSubAccount;
                            BayarKe = resultA.MFBRK.MfbrkBroker;
                        }
                        else
                        {
                            response.SetReturnStatus(DanaPensiunConstants.ERROR_STATUS, "Tidak dapat Disimpan karena Account & Sub Account Investasi belum ada");
                            return response;
                        }

                        Ket = resultA.TRD_RD.Ket;
                    }

                    KetHutPiut = _sharedFunction.ConvertAcn(HutPiut, "A") + " - " + _sharedFunction.ConvertAcn(SubAcn, "S");


                    //generate jurnal per BANK KELUAR

                    var queryB = from trdepoOh in _context.TrdepoRds
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
                        KetBank = _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomAccount, "A") + " - " + _sharedFunction.ConvertAcn(item.MFBNKCOM.MfbnkcomSubAcc, "S");
                        reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);

                        //Validasi sudah digunakan atau belum No. Ref Jurnalnya
                        var resultC = await _context.Gltrhdsums.Where(w => w.GltrhdReffNumb == reff).ToListAsync();

                        foreach (var gl in resultC)
                        {
                            reff = await _sharedFunction.GenerateNewNoJurnal(TypeGL, item.MFBNKCOM.MfbnkcomKodeNoJurnal, request.TanggalBayar, cancellationToken);
                        }

                        AllReff = AllReff + ",'" + reff + "'";

                        //TBC

                        if (item.TRDEPO_OH.Jumlah < 0)
                        {
                            //bank - Debet
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 1, item.MFBNKCOM.MfbnkcomAccount, item.MFBNKCOM.MfbnkcomSubAcc, request.TanggalBayar, KetBank, item.TRDEPO_OH.Jumlah.Value, 0, yr, Period.Value, "DPPK", "", "Arus Kas", "IDR", 1, "DPPK", 1);

                            //Piutang Investasi - Kredit
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 2, HutPiut, SubAcn, request.TanggalBayar, KetHutPiut, 0, item.TRDEPO_OH.Jumlah.Value, yr, Period.Value, "BPI", "", "DPPK", "IDR", 1, "DPPK", 1);
                        }
                        else
                        {
                            //Hutang Investasi - Debet
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 1, HutPiut, SubAcn, request.TanggalBayar, KetHutPiut, item.TRDEPO_OH.Jumlah.Value, 0, yr, Period.Value, "DPPK", "", "DPPK", "IDR", 1, "DPPK", 1);

                            //bank - Kredit
                            await _sharedFunction.UpdateDetailJurnal(cancellationToken, reff, 2, item.MFBNKCOM.MfbnkcomAccount, item.MFBNKCOM.MfbnkcomSubAcc, request.TanggalBayar, KetBank, 0, item.TRDEPO_OH.Jumlah.Value, yr, Period.Value, "DPPK", "", "Arus Kas", "IDR", 1, "DPPK", 1);
                        }

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

                        //update TRLAK
                        if (JualAmt > 0)
                        {
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
                        if (Beban > 0)
                        {
                            var queryF = $"select * from MSRPT where MSRPT_Modul ='BBI'";
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
                        if (BeliAmt > 0)
                        {
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

                                    newTRLAK.TrlakAmount = (resultG.TrlakAmount == null ? 0 : resultG.TrlakAmount) + total;

                                    _context.Trlaks.Add(newTRLAK);
                                    await _context.SaveChangesAsync(cancellationToken);
                                }
                            }
                        }

                    }

                    //update flag Bayar
                    var queryE = $"update TRD_Saham set Bayar=1 where DP='{request.DpCode}' and NoKonfirmasiBroker='{noSurat}'";
                    var resultE = _context.ExecuteSqlRaw(queryE);

                }
