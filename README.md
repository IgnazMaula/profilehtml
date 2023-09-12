Public Function BuatNomorMemo(ByVal tanggal As Date, ByVal jenis As String) As String
On Error GoTo err
Dim str As String, rs As New ADODB.Recordset, Angka As Double
Dim bln As Double, Thn As Double
using System;
using System.Data;
using System.Data.SqlClient;

public static class MemoNumberGenerator
{
    public static string GenerateMemoNumber(DateTime tanggal, string jenis)
    {
        string connectionString = "your_connection_string_here"; // Replace with your actual connection string
        double bln = 0, Thn = 0;
        double Angka;
        
        try
        {
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();

                string query = "SELECT * FROM bulan WHERE periode = @periode";
                using (SqlCommand command = new SqlCommand(query, connection))
                {
                    command.Parameters.AddWithValue("@periode", tanggal.ToString("yyyy/MM"));
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        if (reader.HasRows)
                        {
                            reader.Read();
                            Angka = Convert.ToDouble(reader["nomor"]);
                        }
                        else
                        {
                            Angka = 0;
                            reader.Close();

                            // Insert a new record if it doesn't exist
                            string insertQuery = "INSERT INTO bulan (periode, nomor) VALUES (@periode, @nomor)";
                            using (SqlCommand insertCommand = new SqlCommand(insertQuery, connection))
                            {
                                insertCommand.Parameters.AddWithValue("@periode", tanggal.ToString("yyyy/MM"));
                                insertCommand.Parameters.AddWithValue("@nomor", Angka + 1);
                                insertCommand.ExecuteNonQuery();
                            }
                        }
                    }
                }

                bln = Angka + 1;

                string yearQuery = "SELECT * FROM tahun WHERE tahun = @tahun";
                using (SqlCommand yearCommand = new SqlCommand(yearQuery, connection))
                {
                    yearCommand.Parameters.AddWithValue("@tahun", tanggal.Year);
                    using (SqlDataReader yearReader = yearCommand.ExecuteReader())
                    {
                        if (yearReader.HasRows)
                        {
                            yearReader.Read();
                            Angka = Convert.ToDouble(yearReader["nomor"]);
                        }
                        else
                        {
                            Angka = 0;
                            yearReader.Close();

                            // Insert a new record if it doesn't exist
                            string insertYearQuery = "INSERT INTO tahun (tahun, nomor) VALUES (@tahun, @nomor)";
                            using (SqlCommand insertYearCommand = new SqlCommand(insertYearQuery, connection))
                            {
                                insertYearCommand.Parameters.AddWithValue("@tahun", tanggal.Year);
                                insertYearCommand.Parameters.AddWithValue("@nomor", Angka + 1);
                                insertYearCommand.ExecuteNonQuery();
                            }
                        }
                    }
                }

                Thn = Angka + 1;
            }
        }
        catch (Exception ex)
        {
            // Handle the exception as needed
            Console.WriteLine(ex.Message);
            return "";
        }

        return $"{bln}/{Thn}/{AngkaRomawi(tanggal.Month)}/{tanggal.Year}";
    }

    public static string AngkaRomawi(int angka)
    {
        // Implement your AngkaRomawi function here
        // This function converts an integer to its Roman numeral representation
        // For example, 1 to "I", 2 to "II", etc.
        // You can find existing implementations of this function online.
        // Return the Roman numeral representation of the given number.
    }
}

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
