# PowerShell-BlackJack
Blackjack mini game for windows powershell.
** Copy & Paste in your Powershell terminal **
```
 function Get-Card {
    $values = @(2..10 + 10 + 10 + 11) # J, Q, K = 10, A = 11
    $suits = @("♠", "♥", "♦", "♣")
    $value = $values | Get-Random
    $suit = $suits | Get-Random
    return @{Value=$value; Display="$value$suit"}
}

function Show-Hand {
    param($hand, $owner)
    $sum = Get-Score $hand
    $cards = ($hand | ForEach-Object { $_.Display }) -join ", "
    "$owner's hand: $cards = $sum"
}

function Get-Score {
    param($hand)
    $total = ($hand | ForEach-Object { $_.Value } | Measure-Object -Sum).Sum
    $aces = ($hand | Where-Object { $_.Value -eq 11 }).Count
    while ($total -gt 21 -and $aces -gt 0) {
        $total -= 10
        $aces--
    }
    return $total
}

function Play-Blackjack {
    param([ref]$wins, [ref]$losses, [ref]$ties)
    Clear-Host
    Write-Host "=== PowerShell Blackjack ===" -ForegroundColor Cyan

    $playerHand = @()
    $dealerHand = @()

    # Initial deal
    1..2 | ForEach-Object {
        $playerHand += Get-Card
        $dealerHand += Get-Card
    }

    Write-Host "`nDealer shows: $($dealerHand[0].Display), ?"
    Write-Host (Show-Hand -hand $playerHand -owner "Player")

    # Player's turn
    while ($true) {
        $score = Get-Score $playerHand
        if ($score -gt 21) {
            Write-Host "`nYou bust with $score! Dealer wins." -ForegroundColor Red
            $losses.Value++
            return
        }
        $choice = Read-Host "Hit or Stand? (h/s)"
        if ($choice -eq 'h') {
            $playerHand += Get-Card
            Write-Host (Show-Hand -hand $playerHand -owner "Player")
        } elseif ($choice -eq 's') {
            break
        }
    }

    # Dealer's turn
    Write-Host "`nDealer's turn..."
    Write-Host (Show-Hand -hand $dealerHand -owner "Dealer")
    while ((Get-Score $dealerHand) -lt 17) {
        Start-Sleep -Milliseconds 700
        $dealerHand += Get-Card
        Write-Host (Show-Hand -hand $dealerHand -owner "Dealer")
    }

    $playerScore = Get-Score $playerHand
    $dealerScore = Get-Score $dealerHand

    if ($dealerScore -gt 21) {
        Write-Host "`nDealer busts with $dealerScore! You win!" -ForegroundColor Green
        $wins.Value++
    } elseif ($dealerScore -gt $playerScore) {
        Write-Host "`nDealer wins with $dealerScore vs your $playerScore." -ForegroundColor Yellow
        $losses.Value++
    } elseif ($dealerScore -lt $playerScore) {
        Write-Host "`nYou win with $playerScore vs dealer's $dealerScore!" -ForegroundColor Green
        $wins.Value++
    } else {
        Write-Host "`nIt's a tie at $playerScore." -ForegroundColor Gray
        $ties.Value++
    }
}

# Game loop
$wins = 0
$losses = 0
$ties = 0

do {
    Play-Blackjack -wins ([ref]$wins) -losses ([ref]$losses) -ties ([ref]$ties)
    Write-Host "`nScoreboard: Wins=$wins, Losses=$losses, Ties=$ties" -ForegroundColor Cyan
    $again = Read-Host "`nPlay again? (y/n)"
} while ($again -eq 'y')

Write-Host "`nThanks for playing PowerShell Blackjack!" -ForegroundColor Magenta
```
