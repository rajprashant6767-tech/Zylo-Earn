package com.example.ui.screens

import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.AccountBalance
import androidx.compose.material.icons.filled.AccountBalanceWallet
import androidx.compose.material.icons.filled.CardGiftcard
import androidx.compose.material.icons.filled.CheckCircle
import androidx.compose.material.icons.filled.Info
import androidx.compose.material.icons.filled.MonetizationOn
import androidx.compose.material.icons.filled.Payments
import androidx.compose.material.icons.filled.Shop
import androidx.compose.material3.AlertDialog
import com.example.ui.components.WithdrawalComponent
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.Icon
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.vector.ImageVector
import androidx.compose.ui.platform.testTag
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.data.local.UserProfile
import com.example.ui.theme.CoinGold
import com.example.ui.theme.EmeraldSuccess
import com.example.ui.theme.IndigoLight
import com.example.ui.theme.IndigoPrimary
import com.example.ui.theme.SlateDark
import com.example.ui.theme.SlateMedium
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

data class WithdrawalMethodOption(
    val name: String,
    val icon: ImageVector,
    val inputPlaceholder: String,
    val inputLabel: String,
    val subtitle: String
)

val withdrawalMethods = listOf(
    WithdrawalMethodOption(
        name = "UPI",
        icon = Icons.Default.Payments,
        inputPlaceholder = "e.g. name@okhdfcbank or 9876543210@paytm",
        inputLabel = "UPI ID / VPA",
        subtitle = "Instant Bank Payout"
    ),
    WithdrawalMethodOption(
        name = "Amazon Gift Card",
        icon = Icons.Default.CardGiftcard,
        inputPlaceholder = "e.g. name@gmail.com",
        inputLabel = "Amazon Account Email",
        subtitle = "Digital Voucher Code"
    ),
    WithdrawalMethodOption(
        name = "Google Play Store Gift Card",
        icon = Icons.Default.Shop,
        inputPlaceholder = "e.g. name@gmail.com",
        inputLabel = "Google Account Email",
        subtitle = "Play Store Redeem Code"
    )
)

val denominations = listOf(10, 20, 50, 100, 200)

@Composable
fun WithdrawalScreen(
    user: UserProfile?,
    withdrawalSuccessRef: String?,
    onSubmitWithdrawal: (method: String, rupees: Int, targetAccount: String) -> Unit,
    onDismissSuccess: () -> Unit,
    modifier: Modifier = Modifier
) {
    val today = remember { SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date()) }
    val alreadyWithdrewToday = user?.lastWithdrawalDate == today

    var lastSubmittedRupees by remember { mutableIntStateOf(10) }
    var lastSubmittedMethod by remember { mutableStateOf("UPI") }

    val currentCoins = user?.coins ?: 0

    LazyColumn(
        modifier = modifier
            .fillMaxSize()
            .background(Color(0xFFFAFAFB))
            .padding(horizontal = 16.dp)
            .testTag("withdrawal_screen_content"),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        item {
            Spacer(modifier = Modifier.height(4.dp))
        }

        // Wallet Balance & Conversion Card
        item {
            Card(
                shape = RoundedCornerShape(20.dp),
                colors = CardDefaults.cardColors(containerColor = Color.White),
                elevation = CardDefaults.cardElevation(defaultElevation = 2.dp),
                modifier = Modifier
                    .fillMaxWidth()
                    .border(1.dp, Color(0xFFE2E8F0), RoundedCornerShape(20.dp))
            ) {
                Column(modifier = Modifier.padding(18.dp)) {
                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween,
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column {
                            Text("Available Wallet Balance", fontSize = 12.sp, color = SlateMedium)
                            Row(verticalAlignment = Alignment.CenterVertically) {
                                Icon(
                                    imageVector = Icons.Default.MonetizationOn,
                                    contentDescription = null,
                                    tint = CoinGold,
                                    modifier = Modifier.size(22.dp)
                                )
                                Spacer(modifier = Modifier.width(6.dp))
                                Text(
                                    text = "%,d Coins".format(currentCoins),
                                    fontSize = 22.sp,
                                    fontWeight = FontWeight.Bold,
                                    color = SlateDark
                                )
                            }
                        }

                        // Conversion Badge
                        Box(
                            modifier = Modifier
                                .background(Color(0xFFFEF3C7), RoundedCornerShape(12.dp))
                                .padding(horizontal = 10.dp, vertical = 6.dp)
                        ) {
                            Text(
                                text = "10,000 Coins = ₹10",
                                fontSize = 11.sp,
                                fontWeight = FontWeight.Bold,
                                color = Color(0xFFB45309)
                            )
                        }
                    }

                    Spacer(modifier = Modifier.height(10.dp))

                    Text(
                        text = "Equivalent Real Cash Value: ₹${currentCoins / 1000}",
                        fontSize = 13.sp,
                        fontWeight = FontWeight.SemiBold,
                        color = EmeraldSuccess
                    )
                }
            }
        }

        // Daily Limit Alert
        if (alreadyWithdrewToday) {
            item {
                Card(
                    shape = RoundedCornerShape(16.dp),
                    colors = CardDefaults.cardColors(containerColor = Color(0xFFFEE2E2)),
                    modifier = Modifier
                        .fillMaxWidth()
                        .testTag("daily_limit_reached_card")
                ) {
                    Row(
                        modifier = Modifier.padding(14.dp),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Icon(
                            imageVector = Icons.Default.Info,
                            contentDescription = null,
                            tint = Color(0xFFDC2626),
                            modifier = Modifier.size(24.dp)
                        )
                        Spacer(modifier = Modifier.width(12.dp))
                        Column {
                            Text(
                                text = "Daily Limit Reached",
                                fontSize = 14.sp,
                                fontWeight = FontWeight.Bold,
                                color = Color(0xFFDC2626)
                            )
                            Text(
                                text = "Maximum 1 withdrawal per user per day. Your next withdrawal unlocks tomorrow at 12:00 AM.",
                                fontSize = 12.sp,
                                color = Color(0xFF991B1B)
                            )
                        }
                    }
                }
            }
        }

        // Interactive Withdrawal Component with Payment Details Input & Threshold Validation
        item {
            WithdrawalComponent(
                userCoins = currentCoins,
                isDailyLimitReached = alreadyWithdrewToday,
                onRequestPayout = { method, rupees, target ->
                    lastSubmittedRupees = rupees
                    lastSubmittedMethod = method
                    onSubmitWithdrawal(method, rupees, target)
                }
            )
        }

        item {
            Spacer(modifier = Modifier.height(32.dp))
        }
    }

    // Success Dialog with Reference ID
    if (withdrawalSuccessRef != null) {
        AlertDialog(
            onDismissRequest = onDismissSuccess,
            title = {
                Row(verticalAlignment = Alignment.CenterVertically) {
                    Icon(
                        imageVector = Icons.Default.CheckCircle,
                        contentDescription = null,
                        tint = EmeraldSuccess,
                        modifier = Modifier.size(26.dp)
                    )
                    Spacer(modifier = Modifier.width(8.dp))
                    Text("Withdrawal Submitted!", fontSize = 18.sp, fontWeight = FontWeight.Bold)
                }
            },
            text = {
                Column(
                    horizontalAlignment = Alignment.CenterHorizontally,
                    modifier = Modifier.fillMaxWidth()
                ) {
                    Text(
                        text = "Your withdrawal of ₹$lastSubmittedRupees via $lastSubmittedMethod has been placed successfully and logged in your Transaction History.",
                        fontSize = 13.sp,
                        color = SlateDark,
                        textAlign = TextAlign.Center
                    )
                    Spacer(modifier = Modifier.height(12.dp))
                    Box(
                        modifier = Modifier
                            .background(Color(0xFFF1F5F9), RoundedCornerShape(8.dp))
                            .padding(horizontal = 12.dp, vertical = 6.dp)
                    ) {
                        Text(
                            text = "Ref: $withdrawalSuccessRef",
                            fontSize = 12.sp,
                            fontWeight = FontWeight.Bold,
                            color = IndigoPrimary
                        )
                    }
                    Spacer(modifier = Modifier.height(8.dp))
                    Text(
                        text = "🔔 A confirmation push notification has been sent to your device with this reference ID.",
                        fontSize = 11.sp,
                        color = SlateMedium,
                        textAlign = TextAlign.Center
                    )
                }
            },
            confirmButton = {
                Button(
                    onClick = onDismissSuccess,
                    colors = ButtonDefaults.buttonColors(containerColor = IndigoPrimary)
                ) {
                    Text("Done")
                }
            }
        )
    }
}
