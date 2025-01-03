using System;
using System.Collections.Generic;
using System.Media;
using System.Threading;

namespace Project
{
    internal class Program
    {
        static int ekranGenisligi = 25;
        static int ekranYuksekligi = 15;
        static int yilanX;
        static int yilanY;
        static int yemekX;
        static int yemekY;
        static int puan;
        static bool oyunBitti;
        static List<(int x, int y)> yilanGovde = new List<(int x, int y)>();
        static List<(int x, int y)> engeller = new List<(int x, int y)>();
        static string yon;
        static int offsetX = 0;
        static int offsetY = 0;
        static int yilanHizi = 150;
        static SoundPlayer yemekPlayer = new SoundPlayer("sounds/eat.wav");
        static SoundPlayer kaybetPlayer = new SoundPlayer("sounds/lose.wav");
        static SoundPlayer bonusPlayer = new SoundPlayer("sounds/bonus.wav");
        static string oyuncuAdi;
        static string oyuncuYas;
        static string oyuncuUlke;
        static string oyuncuSehir;
        static int ozelYemekX;
        static int ozelYemekY;
        static bool ozelYemekAktif = false;
        static int ozelYemekSuresi = 0;
        static int ozelYemekSayac = 0;
        static int seviye = 1;
        static int maxSeviye = 5;
        static int[] seviyePuanlari = { 20, 40, 60, 80, 100 };
        static DateTime ozelYemekMesajZamani;

        static void Main(string[] args)
        {
            Console.CursorVisible = false;

            while (true)
            {
                AnaMenu();
            }
        }

        static void AnaMenu()
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Yılan Oyununa hos geldiniz:");
            Console.WriteLine("(1) Oyunu Başlat");
            Console.WriteLine("(2) Seviyeyi Sıfırla ve Baştan Başla");
            Console.WriteLine("(3) Kuralları Görüntüle");
            Console.WriteLine("(4) Oyuncu Bilgileri");
            Console.WriteLine("(5) Çıkış");
            var secim = Console.ReadKey(true).Key;

            if (secim == ConsoleKey.D1 || secim == ConsoleKey.NumPad1)
            {
                if (string.IsNullOrEmpty(oyuncuAdi))
                {
                    OyuncuBilgileriniAl();
                }
                OyunuBaslat();
                OyunuOyna();
                OyunBittiEkrani();
            }
            else if (secim == ConsoleKey.D2 || secim == ConsoleKey.NumPad2)
            {
                OyuncuBilgileriniAl();
                SifirdanBaslat();
            }
            else if (secim == ConsoleKey.D3 || secim == ConsoleKey.NumPad3)
            {
                KurallariGoster();
            }
            else if (secim == ConsoleKey.D4 || secim == ConsoleKey.NumPad4)
            {
                OyuncuBilgileriniGoster();
            }
            else if (secim == ConsoleKey.D5 || secim == ConsoleKey.NumPad5)
            {
                Environment.Exit(0);
            }
        }

        static void OyuncuBilgileriniAl()
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Merhaba! Lütfen adınızı girin:");
            Console.ForegroundColor = ConsoleColor.White;
            oyuncuAdi = Console.ReadLine();

            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Yaşınızı girin:");
            Console.ForegroundColor = ConsoleColor.White;
            oyuncuYas = Console.ReadLine();

            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Ülkenizi girin:");
            Console.ForegroundColor = ConsoleColor.White;
            oyuncuUlke = Console.ReadLine();

            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Şehrinizi girin:");
            Console.ForegroundColor = ConsoleColor.White;
            oyuncuSehir = Console.ReadLine();
        }

        static void OyuncuBilgileriniGoster()
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Oyuncu Bilgileri:");
            Console.WriteLine($"Ad: {oyuncuAdi}");
            Console.WriteLine($"Yaş: {oyuncuYas}");
            Console.WriteLine($"Ülke: {oyuncuUlke}");
            Console.WriteLine($"Şehir: {oyuncuSehir}");
            Console.WriteLine("Devam etmek için 1'e basın...");
            Console.ReadKey(true);
        }

        static void KurallariGoster()
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine("Kurallar:");
            Console.WriteLine("- Hareket etmek için W (Yukarı), A (Sol), S (Aşağı), D (Sağ) tuşlarını veya yön tuşlarını (Ok Tuşları) kullanabilirsiniz.");
            Console.WriteLine("- Oyun beş seviyeden oluşuyor ve her seviyede yılanın hızı artıyor.");
            Console.WriteLine("- Yemek (*) yiyerek puan kazanın.");
            Console.WriteLine("- Özel Yemek (B) 3 puan kazandırır ve yılanı büyütür,Özel yemek belirli bir süre sonra kaybolur.");
            Console.WriteLine("- Oyun 100 puana ulaşıldığında tamamlanır.");
            Console.WriteLine("- Seviye 3'ten itibaren engeller eklenecek.");
            Console.WriteLine("- Oyuncu seviyeleri geçtikçe puanlarını biriktirir, Kaybettiğinizde puan sıfırlanır ve seviyeleri tekrar tamamlamak için puanları yeniden toplamanız gerekir");
            Console.WriteLine("- Puanları topla ve oyunun sonunda ödülü kazan.Bol şanslar dileriz");
            Console.WriteLine("Devam etmek için 1'e basın...");
            Console.ReadKey(true);
        }

        static void OyunuBaslat()
        {
            yilanX = ekranGenisligi / 2;
            yilanY = ekranYuksekligi / 2;
            oyunBitti = false;
            yon = "SAG";
            yilanGovde.Clear();
            engeller.Clear();
            if (seviye >= 3) EngellerOlustur();
            offsetX = (Console.WindowWidth - ekranGenisligi - 2) / 2;
            offsetY = (Console.WindowHeight - ekranYuksekligi - 5) / 2;
            YemekOlustur();
        }

        static void SifirdanBaslat()
        {
            seviye = 1;
            puan = 0;
            OyunuBaslat();
            OyunuOyna();
            OyunBittiEkrani();
        }

        static void OyunuOyna()
        {
            Thread inputThread = new Thread(KlavyeGirdisiOku);
            inputThread.Start();

            while (!oyunBitti)
            {
                EkraniCiz();
                YilaniHareketEttir();
                CarpismaKontrol();

                if (!ozelYemekAktif)
                {
                    ozelYemekSayac++;
                    if (ozelYemekSayac >= 200)
                    {
                        OzelYemekOlustur();
                        ozelYemekSayac = 0;
                    }
                }

                if (ozelYemekAktif)
                {
                    ozelYemekSuresi--;
                    if (ozelYemekSuresi <= 0)
                    {
                        ozelYemekAktif = false;
                    }
                }

                if (puan >= seviyePuanlari[seviye - 1] && seviye < maxSeviye)
                {
                    SeviyeArttir();
                }

                if (seviye == maxSeviye && puan >= seviyePuanlari[maxSeviye - 1])
                {
                    OyunTamamlandi();
                }

                Thread.Sleep(yilanHizi);
            }

            inputThread.Abort();
        }

        static void SeviyeArttir()
        {
            seviye++;
            yilanHizi = Math.Max(50, yilanHizi - 20);
            yilanGovde.Clear();
            yilanX = ekranGenisligi / 2;
            yilanY = ekranYuksekligi / 2;
            yon = "SAG"; 
            if (seviye >= 3) EngellerOlustur();
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine($"Tebrikler {oyuncuAdi}, {seviye}. seviyeye ulaştınız!");
            Console.WriteLine($"Skor: {puan}, Geçmek için gereken puan: {seviyePuanlari[seviye - 1]}");
            Console.WriteLine("Devam etmek için S tuşuna basın...");
            while (Console.ReadKey(true).Key != ConsoleKey.S) { }
        }

        static void OyunTamamlandi()
        {
            oyunBitti = true;
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine($"Tebrikler {oyuncuAdi}! Oyunu başarıyla tamamladınız!");
            Console.WriteLine("Azminiz ve beceriniz sayesinde bu başarıyı elde ettiniz. Harika bir iş çıkardınız, bravo!");
            Console.WriteLine("Ana menüye dönmek için 1'e basın...");
            Console.ReadKey(true);
            AnaMenu();
        }

        static void OzelYemekOlustur()
        {
            Random random = new Random();
            int minDistance = 5;
            bool validPosition;

            do
            {
                validPosition = true;
                ozelYemekX = random.Next(0, ekranGenisligi);
                ozelYemekY = random.Next(0, ekranYuksekligi);

                int deltaX = Math.Abs(ozelYemekX - yilanX);
                int deltaY = Math.Abs(ozelYemekY - yilanY);

                if (deltaX < minDistance || deltaY < minDistance)
                {
                    validPosition = false;
                }

                if (yilanGovde.Contains((ozelYemekX, ozelYemekY)) || engeller.Contains((ozelYemekX, ozelYemekY)))
                {
                    validPosition = false;
                }
            } while (!validPosition);

            ozelYemekAktif = true;
            ozelYemekSuresi = 100;
            ozelYemekMesajZamani = DateTime.Now.AddSeconds(5);
        }

        static void EngellerOlustur()
        {
            engeller.Clear();
            Random random = new Random();
            int engelSayisi = seviye == 3 ? 5 : seviye == 4 ? 8 : 12;

            for (int i = 0; i < engelSayisi; i++)
            {
                int engelX = random.Next(0, ekranGenisligi);
                int engelY = random.Next(0, ekranYuksekligi);
                if (!engeller.Contains((engelX, engelY)) && (engelX != yemekX || engelY != yemekY))
                {
                    engeller.Add((engelX, engelY));
                }
            }
        }

        static void EkraniCiz()
        {
            Console.SetCursorPosition(0, 0);

            Console.SetCursorPosition(offsetX, offsetY);
            Console.ForegroundColor = ConsoleColor.Yellow;
            for (int i = 0; i < ekranGenisligi + 2; i++)
                Console.Write("H");
            Console.WriteLine();

            for (int y = 0; y < ekranYuksekligi; y++)
            {
                Console.SetCursorPosition(offsetX, offsetY + y + 1);
                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.Write("H");
                for (int x = 0; x < ekranGenisligi; x++)
                {
                    if (x == yilanX && y == yilanY)
                    {
                        Console.ForegroundColor = ConsoleColor.Green;
                        Console.Write("O");
                    }
                    else if (yilanGovde.Contains((x, y)))
                    {
                        Console.ForegroundColor = ConsoleColor.Green;
                        Console.Write("o");
                    }
                    else if (x == yemekX && y == yemekY)
                    {
                        Console.ForegroundColor = ConsoleColor.DarkRed;
                        Console.Write("*");
                    }
                    else if (ozelYemekAktif && x == ozelYemekX && y == ozelYemekY)
                    {
                        Console.ForegroundColor = ConsoleColor.Magenta;
                        Console.Write("B");
                    }
                    else if (engeller.Contains((x, y)))
                    {
                        Console.ForegroundColor = ConsoleColor.Red;
                        Console.Write("X");
                    }
                    else
                    {
                        Console.Write(" ");
                    }
                }
                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.Write("H");
                Console.WriteLine();
            }

            Console.SetCursorPosition(offsetX, offsetY + ekranYuksekligi + 1);
            for (int i = 0; i < ekranGenisligi + 2; i++)
                Console.Write("H");

            Console.SetCursorPosition(offsetX, offsetY + ekranYuksekligi + 3);
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine($"Puan: {puan}");
            Console.SetCursorPosition(offsetX, offsetY + ekranYuksekligi + 4);
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine($"Seviye: {seviye}, Geçmek için gereken: {seviyePuanlari[seviye - 1]}");

            if (ozelYemekAktif && DateTime.Now <= ozelYemekMesajZamani)
            {
                Console.SetCursorPosition(offsetX, offsetY + ekranYuksekligi + 5);
                Console.ForegroundColor = ConsoleColor.Magenta;
                Console.WriteLine("Özel Yemek Göründü! Hemen Alın!");
            }
            else if (DateTime.Now > ozelYemekMesajZamani) 
            {
                Console.SetCursorPosition(offsetX, offsetY + ekranYuksekligi + 5);
                Console.Write(new string(' ', Console.WindowWidth)); 
            }
        }

        static void YilaniHareketEttir()
        {
            yilanGovde.Insert(0, (yilanX, yilanY));

            switch (yon)
            {
                case "YUKARI":
                    yilanY--;
                    break;
                case "ASAGI":
                    yilanY++;
                    break;
                case "SOL":
                    yilanX--;
                    break;
                case "SAG":
                    yilanX++;
                    break;
            }

            if (yilanX == yemekX && yilanY == yemekY)
            {
                puan++;
                YemekSesiCal();
                YemekOlustur();
            }
            else
            {
                if (yilanGovde.Count > 0)
                    yilanGovde.RemoveAt(yilanGovde.Count - 1);
            }

            if (ozelYemekAktif && yilanX == ozelYemekX && yilanY == ozelYemekY)
            {
                puan += 3;
                for (int i = 0; i < 3; i++)
                {
                    if (yilanGovde.Count > 0)
                    {
                        yilanGovde.Add(yilanGovde[yilanGovde.Count - 1]);
                    }
                    else
                    {
                        yilanGovde.Add((yilanX, yilanY));
                    }
                }
                OzelYemekSesiCal();
                ozelYemekAktif = false;
            }
        }

        static void YemekOlustur()
        {
            Random random = new Random();
            yemekX = random.Next(0, ekranGenisligi);
            yemekY = random.Next(0, ekranYuksekligi);

            while (yilanGovde.Contains((yemekX, yemekY)) || (yilanX == yemekX && yilanY == yemekY) || engeller.Contains((yemekX, yemekY)))
            {
                yemekX = random.Next(0, ekranGenisligi);
                yemekY = random.Next(0, ekranYuksekligi);
            }
        }

        static void CarpismaKontrol()
        {
            if (yilanX < 0 || yilanX >= ekranGenisligi || yilanY < 0 || yilanY >= ekranYuksekligi)
            {
                oyunBitti = true;
                KaybetmeSesiCal();
            }

            if (yilanGovde.Contains((yilanX, yilanY)) || engeller.Contains((yilanX, yilanY)))
            {
                oyunBitti = true;
                KaybetmeSesiCal();
            }
        }

        static void KlavyeGirdisiOku()
        {
            while (!oyunBitti)
            {
                var key = Console.ReadKey(true).Key;

                if ((key == ConsoleKey.W || key == ConsoleKey.UpArrow) && yon != "ASAGI")
                    yon = "YUKARI";
                else if ((key == ConsoleKey.S || key == ConsoleKey.DownArrow) && yon != "YUKARI")
                    yon = "ASAGI";
                else if ((key == ConsoleKey.A || key == ConsoleKey.LeftArrow) && yon != "SAG")
                    yon = "SOL";
                else if ((key == ConsoleKey.D || key == ConsoleKey.RightArrow) && yon != "SOL")
                    yon = "SAG";
            }
        }

        static void OyunBittiEkrani()
        {
            Console.Clear();
            Console.ForegroundColor = ConsoleColor.Blue;
            Console.WriteLine($"Oyun Bitti, {oyuncuAdi}!");
            Console.WriteLine($"Son Puan: {puan}");
            Console.WriteLine("(1) Bu Seviyeyi Yeniden Başlat.");
            Console.WriteLine("(2) Ana Menüye Dön.");
            var secim = Console.ReadKey(true).Key;

            if (secim == ConsoleKey.D1 || secim == ConsoleKey.NumPad1)
            {
                puan = 0;
                OyunuBaslat();
                OyunuOyna();
                OyunBittiEkrani();
            }
            else if (secim == ConsoleKey.D2 || secim == ConsoleKey.NumPad2)
            {
                AnaMenu();
            }
        }

        static void YemekSesiCal()
        {
            try { yemekPlayer.Play(); }
            catch { Console.Beep(600, 150); }
        }

        static void KaybetmeSesiCal()
        {
            try { kaybetPlayer.Play(); }
            catch { Console.Beep(200, 300); }
        }

        static void OzelYemekSesiCal()
        {
            try { bonusPlayer.Play(); }
            catch { Console.Beep(800, 200); }
        }
    }
}
