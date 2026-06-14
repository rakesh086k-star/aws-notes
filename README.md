Get-AzReservation |
ForEach-Object {
    Get-AzReservationSummary `
        -ReservationOrderId $_.ReservationOrderId `
        -ReservationId $_.Id
}