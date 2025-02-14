# EA Timer Trade For XAUUSD

//+------------------------------------------------------------------+
//| Expert Advisor untuk XAUUSD (M15)                               |
//| Entry pukul 20:15 WIB, exit setelah 23 jam 45 menit berjalan   |
//| LotSize otomatis mengikuti equity                               |
//+------------------------------------------------------------------+
#include <Trade/Trade.mqh>
CTrade trade;

// Parameter input
input double RiskPercentage = 1.0; // Persentase risiko terhadap equity
input string TradingSymbol = "XAUUSD";

// Fungsi untuk menghitung lot berdasarkan equity
double CalculateLotSize() {
   double equity = AccountInfoDouble(ACCOUNT_EQUITY);
   double lotSize = (equity * (RiskPercentage / 100.0)) / 1000.0; // Asumsi 1000 USD per 1 lot
   return NormalizeDouble(lotSize, 2);
}

// Fungsi untuk mendapatkan waktu saat order dibuka
datetime GetLastOrderOpenTime() {
   for (int i = PositionsTotal() - 1; i >= 0; i--) {
      if (PositionSelect(i) && PositionSymbol() == TradingSymbol) {
         return PositionGetInteger(POSITION_TIME);
      }
   }
   return 0; // Tidak ada order aktif
}

void OnTick() {
   static bool hasTraded = false;
   datetime serverTime = TimeCurrent();
   int hour = TimeHour(serverTime);
   int minute = TimeMinute(serverTime);

   // Entry pada 20:15 WIB jika belum ada trade hari ini
   if (hour == 20 && minute == 15 && !hasTraded) {
      double lotSize = CalculateLotSize();
      double lastOrderType = GetLastOrderOpenTime();
      if (lastOrderType > 0) {
         trade.Sell(lotSize, TradingSymbol, 0, 0, 0);
      } else {
         trade.Buy(lotSize, TradingSymbol, 0, 0, 0);
      }
      hasTraded = true;
   }

   // Exit setelah 23 jam 45 menit berjalan
   for (int i = PositionsTotal() - 1; i >= 0; i--) {
      if (PositionSelect(i) && PositionSymbol() == TradingSymbol) {
         datetime openTime = PositionGetInteger(POSITION_TIME);
         if (serverTime - openTime >= 23 * 3600 + 45 * 60) {
            trade.PositionClose(PositionGetInteger(POSITION_TICKET));
         }
      }
   }
}
