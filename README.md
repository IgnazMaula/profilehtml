var query = from x in
                (from y in
                    (from b in context.TRD_OH_SBP
                     join m in context.MOBL on b.TRD_OH_Kode equals m.MOBL_Code
                     where m.MOBL_MaturityDate.Year == 2023 && m.MOBL_MaturityDate.Month == 7
                     select new
                     {
                         b.ApproveJT,
                         b.TRD_OH_Kode,
                         b.Nominal,
                         m.MOBL_IssuedDate,
                         m.MOBL_MaturityDate,
                         nominalJual = (from s in context.TRD_OH_SBP_Sell
                                        where s.noBeli == b.TRD_OH_NO
                                        select (decimal?)s.Nominal).Sum() ?? 0
                     })
                 select new
                 {
                     y.ApproveJT,
                     y.TRD_OH_Kode,
                     y.MOBL_IssuedDate,
                     y.MOBL_MaturityDate,
                     Nominal = y.nominalJual == 0 ? y.Nominal : y.Nominal + y.nominalJual
                 })
            group x by new
            {
                x.ApproveJT,
                x.TRD_OH_Kode,
                x.MOBL_IssuedDate,
                x.MOBL_MaturityDate
            } into g
            select new
            {
                g.Key.ApproveJT,
                g.Key.TRD_OH_Kode,
                Nominal = g.Sum(x => x.Nominal),
                g.Key.MOBL_IssuedDate,
                g.Key.MOBL_MaturityDate
            }
            orderby g.Key.MOBL_MaturityDate
            select new
            {
                g.Key.ApproveJT,
                g.Key.TRD_OH_Kode,
                g.Sum(x => x.Nominal),
                g.Key.MOBL_IssuedDate,
                g.Key.MOBL_MaturityDate
            };

var result = query.ToList();
