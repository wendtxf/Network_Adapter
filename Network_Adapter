$ErrorActionPreference = 'Stop'

function Write-Section {
    param([string]$Title)

    Write-Host ''
    Write-Host ('  {0}' -f $Title) -ForegroundColor Cyan
    Write-Host ('  ' + ('-' * 58)) -ForegroundColor DarkCyan
}

Clear-Host
Write-Host ('=' * 62) -ForegroundColor DarkCyan
Write-Host '                 GERENCIADOR DE REDE' -ForegroundColor Cyan
Write-Host '          Visualizar e alterar adaptadores' -ForegroundColor Gray
Write-Host ('=' * 62) -ForegroundColor DarkCyan

function Test-IsAdministrator {
    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = New-Object Security.Principal.WindowsPrincipal($identity)
    return $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
}

if (-not (Test-IsAdministrator)) {
    Write-Host ''
    Write-Host '[ERRO] Execute este script como administrador.' -ForegroundColor Red
    exit 1
}

foreach ($commandName in @('Get-NetAdapter', 'Enable-NetAdapter', 'Disable-NetAdapter')) {
    if (-not (Get-Command $commandName -ErrorAction SilentlyContinue)) {
        Write-Host ''
        Write-Host "[ERRO] O comando '$commandName' não está disponível neste Windows." -ForegroundColor Red
        exit 1
    }
}

try {
    $adapters = @(Get-NetAdapter -ErrorAction Stop | Sort-Object -Property Name)
    if ($adapters.Count -eq 0) {
        Write-Host ''
        Write-Host '[INFO] Nenhum adaptador de rede foi encontrado.' -ForegroundColor Yellow
        exit 0
    }

    Write-Section 'ADAPTADORES DISPONÍVEIS'
    Write-Host ('  {0,-5} {1,-24} {2,-16} {3}' -f 'Nº', 'NOME', 'ESTADO', 'DESCRIÇÃO') -ForegroundColor White
    for ($i = 0; $i -lt $adapters.Count; $i++) {
        $adapter = $adapters[$i]
        $statusColor = if ($adapter.Status -eq 'Up') { 'Green' } elseif ($adapter.Status -eq 'Disabled') { 'Red' } else { 'Yellow' }
        Write-Host ('  {0,-5} {1,-24} ' -f "[$($i + 1)]", $adapter.Name) -NoNewline
        Write-Host ('{0,-16} ' -f $adapter.Status) -ForegroundColor $statusColor -NoNewline
        Write-Host $adapter.InterfaceDescription -ForegroundColor Gray
    }

    Write-Section 'SELEÇÃO'
    $adapterChoice = Read-Host '  Número do adaptador'
    if ($adapterChoice -notmatch '^\d+$') {
        throw 'Informe um número válido da lista de adaptadores.'
    }

    $adapterIndex = [int]$adapterChoice - 1
    if ($adapterIndex -lt 0 -or $adapterIndex -ge $adapters.Count) {
        throw 'O número informado não corresponde a um adaptador da lista.'
    }

    $adapter = $adapters[$adapterIndex]

    Write-Section 'OPERAÇÃO'
    Write-Host "  Adaptador selecionado: $($adapter.Name)" -ForegroundColor White
    Write-Host "  Estado atual: $($adapter.Status)" -ForegroundColor Gray
    Write-Host ''
    Write-Host '  [1] Ativar' -ForegroundColor Green
    Write-Host '  [2] Desativar' -ForegroundColor Red
    $action = Read-Host '  Escolha'
    if ($action -notin @('1', '2')) {
        throw 'Opção inválida. Escolha 1 para ativar ou 2 para desativar.'
    }

    if ($action -eq '1' -and $adapter.Status -ne 'Disabled') {
        Write-Host ''
        Write-Host "  O adaptador já está habilitado (estado: $($adapter.Status))." -ForegroundColor Yellow
        exit 0
    }
    if ($action -eq '2' -and $adapter.Status -eq 'Disabled') {
        Write-Host ''
        Write-Host '  O adaptador já está desabilitado.' -ForegroundColor Yellow
        exit 0
    }

    if ($action -eq '2') {
        Write-Host ''
        Write-Host '  ATENÇÃO: desativar pode interromper a conexão' -ForegroundColor Yellow
        Write-Host '  e o acesso remoto a esta máquina.' -ForegroundColor Yellow
        $confirmation = Read-Host "Confirma desativar '$($adapter.Name)'? (S/N)"
        if ($confirmation -notmatch '^(S|SIM)$') {
            Write-Host '  Operação cancelada.' -ForegroundColor Gray
            exit 0
        }

        Disable-NetAdapter -Name $adapter.Name -Confirm:$false -ErrorAction Stop
        $expectedStatus = 'Disabled'
        $successMessage = "Adaptador '$($adapter.Name)' desativado."
    }
    else {
        $confirmation = Read-Host "Confirma ativar '$($adapter.Name)'? (S/N)"
        if ($confirmation -notmatch '^(S|SIM)$') {
            Write-Host '  Operação cancelada.' -ForegroundColor Gray
            exit 0
        }

        Enable-NetAdapter -Name $adapter.Name -Confirm:$false -ErrorAction Stop
        $expectedStatus = 'Enabled'
        $successMessage = "Adaptador '$($adapter.Name)' ativado."
    }

    Start-Sleep -Seconds 2
    $updatedAdapter = Get-NetAdapter -Name $adapter.Name -ErrorAction Stop
    $operationSucceeded = if ($expectedStatus -eq 'Disabled') {
        $updatedAdapter.Status -eq 'Disabled'
    }
    else {
        $updatedAdapter.Status -ne 'Disabled'
    }

    if (-not $operationSucceeded) {
        throw "O estado esperado não foi confirmado. Estado atual: $($updatedAdapter.Status)."
    }

    Write-Section 'RESULTADO'
    if ($expectedStatus -eq 'Enabled') {
        Write-Host "  Adaptador '$($adapter.Name)' habilitado." -ForegroundColor Green
        Write-Host "  Estado da conexão: $($updatedAdapter.Status)" -ForegroundColor Gray
        if ($updatedAdapter.Status -eq 'Disconnected') {
            Write-Host '  Sem link de rede no momento. Verifique o cabo, o ponto de acesso ou a rede Wi-Fi.' -ForegroundColor Yellow
        }
    }
    else {
        Write-Host "  $successMessage" -ForegroundColor Green
        Write-Host "  Estado confirmado: $($updatedAdapter.Status)" -ForegroundColor Green
    }
}
catch {
    Write-Host ''
    Write-Host "  [ERRO] $($_.Exception.Message)" -ForegroundColor Red
    exit 1
}
