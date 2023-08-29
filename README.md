using System;
using System.Data.Linq;
using System.Linq;

class Program
{
    static void UpdateHarian(string tKode, string tType, DateTime tDate, double tNominal, double tKurs, string tJenisTransaksi, string tdp, string tPihak, string tnama)
    {
        try
        {
            DataContext context = new DataContext(/* provide your data context here */);

            if (string.IsNullOrEmpty(tPihak))
            {
                var pihakQuery = from p in context.MFPIHAK
                                 where p.MFPIHAK_Nama == tnama
                                 select p.MFPIHAK_Kode;

                tPihak = pihakQuery.FirstOrDefault();
            }

            var deleteQuery = context.AVGD
                .Where(a => a.AVGD_DPCode == tdp && a.AVGD_Kode == tKode && a.AVGD_Tanggal > tDate)
                .Delete();

            var maxDateQuery = from a in context.AVGD
                               where a.AVGD_DPCode == tdp && a.AVGD_Kode == tKode
                               group a by new { a.AVGD_DPCode, a.AVGD_Kode } into g
                               select new
                               {
                                   MaxDate = g.Max(a => a.AVGD_Tanggal)
                               };

            var rs = context.AVGD
                .Where(a => a.AVGD_DPCode == tdp && a.AVGD_Kode == tKode && a.AVGD_Tanggal == maxDateQuery.FirstOrDefault().MaxDate)
                .FirstOrDefault();

            if (rs == null)
            {
                AVGD newRecord = new AVGD
                {
                    AVGD_Kode = tKode,
                    avgd_type = tType,
                    avgd_trans = tJenisTransaksi,
                    AVGD_Tanggal = tDate,
                    avgd_saldoawal = 0,
                    avgd_unit = tNominal,
                    AVGD_SaldoAkhir = tNominal,
                    avgd_average = Convert.ToDouble(tKurs),
                    AVGD_DPCode = tdp,
                    avgd_pihak = tPihak,
                    avgd_nama = tnama
                };

                if (tType == "OBL" || tType == "HTM" || tType == "SBP")
                {
                    newRecord.avgd_Total = Convert.ToDouble(tNominal) * Convert.ToDouble(tKurs) / 100;
                }
                else
                {
                    newRecord.avgd_Total = Convert.ToDouble(tNominal) * Convert.ToDouble(tKurs);
                }

                context.AVGD.InsertOnSubmit(newRecord);
                context.SubmitChanges();
            }
            else
            {
                // Logic for updating existing record
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }

    static void Main()
    {
        // Call your function here with appropriate parameter values
    }
}
