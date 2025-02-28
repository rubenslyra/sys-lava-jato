# Sistema de Gerenciamento de Lava-Jato "AutoClean"

## Estrutura do Projeto

```
autoClean/
│
├── public/                 # Pasta pública (acessível pelo navegador)
│   ├── index.php           # Página principal/dashboard
│   ├── agendamento.php     # Página de criação de agendamentos
│   ├── admin.php           # Painel de administração
│   ├── configurar.php      # Configuração inicial do sistema
│   ├── assets/             # Recursos estáticos
│   │   ├── css/
│   │   │   └── styles.css
│   │   ├── js/
│   │   │   └── scripts.js
│   │   └── img/
│   │       └── logo.png
│
├── src/                    # Código-fonte da aplicação
│   ├── Services/           # Serviços da aplicação
│   │   ├── AgendamentoService.php
│   │   ├── ClienteService.php
│   │   ├── ServicoService.php
│   │   ├── NotificacaoService.php
│   │   └── IdGeneratorService.php
│   │
│   ├── Commands/           # Comandos (padrão CQRS)
│   │   ├── CriarAgendamentoCommand.php
│   │   ├── AtualizarAgendamentoCommand.php
│   │   ├── CancelarAgendamentoCommand.php
│   │   └── EnviarNotificacaoCommand.php
│   │
│   ├── Queries/            # Consultas (padrão CQRS)
│   │   ├── BuscarAgendamentoQuery.php
│   │   ├── ListarAgendamentosQuery.php
│   │   ├── VerificarDisponibilidadeQuery.php
│   │   └── ListarClientesQuery.php
│   │
│   ├── Models/             # Modelos de dados
│   │   ├── Agendamento.php
│   │   ├── Cliente.php
│   │   ├── Servico.php
│   │   ├── Funcionario.php
│   │   └── Notificacao.php
│   │
│   ├── Handlers/           # Manipuladores de comandos
│   │   ├── CriarAgendamentoHandler.php
│   │   ├── AtualizarAgendamentoHandler.php
│   │   ├── CancelarAgendamentoHandler.php
│   │   └── EnviarNotificacaoHandler.php
│   │
│   ├── Repositories/       # Repositórios de dados
│   │   ├── AgendamentoRepository.php
│   │   ├── ClienteRepository.php
│   │   ├── ServicoRepository.php
│   │   └── FuncionarioRepository.php
│   │
│   ├── Utils/              # Utilitários
│   │   ├── Logger.php
│   │   ├── DateTimeHelper.php
│   │   └── Mailer.php
│   │
│   └── Config/             # Configurações
│       ├── Database.php
│       ├── Email.php
│       └── App.php
│
├── templates/              # Templates HTML
│   ├── header.php
│   ├── footer.php
│   ├── sidebar.php
│   ├── modals/
│   │   ├── confirmacao.php
│   │   └── erro.php
│   └── emails/
│       ├── confirmacao_agendamento.html
│       ├── lembrete_agendamento.html
│       └── cancelamento_agendamento.html
│
├── data/                   # Dados persistidos (se usar arquivos)
│   ├── agendamentos.json
│   ├── clientes.json
│   └── servicos.json
│
├── vendor/                 # Dependências (Composer)
│
├── composer.json           # Configuração do Composer
├── .htaccess               # Configurações do Apache
├── config.php              # Configurações gerais
└── bootstrap.php           # Inicialização da aplicação
```

## Arquivos Principais

### 1. Config Central (`config.php`)

```php
<?php
// Configurações gerais do sistema
define('APP_NAME', 'AutoClean Lava-Jato');
define('APP_VERSION', '1.0.0');
define('APP_URL', 'http://localhost/autoClean');

// Caminhos do sistema
define('ROOT_PATH', __DIR__);
define('PUBLIC_PATH', ROOT_PATH . '/public');
define('SRC_PATH', ROOT_PATH . '/src');
define('TEMPLATES_PATH', ROOT_PATH . '/templates');
define('DATA_PATH', ROOT_PATH . '/data');

// Configurações de banco de dados (ou arquivos)
define('DB_TYPE', 'file'); // 'file' ou 'mysql'
define('DB_HOST', 'localhost');
define('DB_NAME', 'autoclean');
define('DB_USER', 'root');
define('DB_PASS', '');

// Configurações do Mailtrap
define('MAIL_HOST', 'smtp.mailtrap.io');
define('MAIL_PORT', 2525);
define('MAIL_USERNAME', 'seu_username');
define('MAIL_PASSWORD', 'sua_senha');
define('MAIL_FROM', 'contato@autoclean.com');
define('MAIL_FROM_NAME', 'AutoClean Lava-Jato');

// Configurações de serviços
$TIPOS_SERVICOS = [
    'basico' => [
        'nome' => 'Lavagem Básica',
        'preco' => 40.00,
        'duracao' => 30, // minutos
        'descricao' => 'Lavagem externa, aspiração e limpeza interna básica'
    ],
    'completa' => [
        'nome' => 'Lavagem Completa',
        'preco' => 70.00,
        'duracao' => 60, // minutos
        'descricao' => 'Lavagem externa, aspiração, limpeza interna completa, aplicação de silicone'
    ],
    'premium' => [
        'nome' => 'Lavagem Premium',
        'preco' => 120.00,
        'duracao' => 120, // minutos
        'descricao' => 'Lavagem completa com polimento, cera, limpeza de motor e higienização interna'
    ],
    'polimento' => [
        'nome' => 'Polimento',
        'preco' => 150.00,
        'duracao' => 180, // minutos
        'descricao' => 'Polimento completo da pintura, remoção de arranhões leves'
    ]
];

// Horários de funcionamento
$HORARIOS_FUNCIONAMENTO = [
    '1' => ['08:00', '18:00'], // Segunda
    '2' => ['08:00', '18:00'], // Terça
    '3' => ['08:00', '18:00'], // Quarta
    '4' => ['08:00', '18:00'], // Quinta
    '5' => ['08:00', '18:00'], // Sexta
    '6' => ['08:00', '16:00'], // Sábado
    '0' => null                // Domingo (fechado)
];

// Status dos agendamentos
$STATUS_AGENDAMENTO = [
    'pendente' => [
        'nome' => 'Pendente',
        'cor' => 'warning',
        'icone' => 'fas fa-clock'
    ],
    'confirmado' => [
        'nome' => 'Confirmado',
        'cor' => 'primary',
        'icone' => 'fas fa-check'
    ],
    'em_andamento' => [
        'nome' => 'Em Andamento',
        'cor' => 'info',
        'icone' => 'fas fa-sync-alt fa-spin'
    ],
    'concluido' => [
        'nome' => 'Concluído',
        'cor' => 'success',
        'icone' => 'fas fa-check-circle'
    ],
    'cancelado' => [
        'nome' => 'Cancelado',
        'cor' => 'danger',
        'icone' => 'fas fa-times-circle'
    ]
];
```

### 2. Inicialização da Aplicação (`bootstrap.php`)

```php
<?php
// Inicializa a aplicação e gerencia dependências
require_once __DIR__ . '/config.php';
require_once __DIR__ . '/vendor/autoload.php';

// Iniciar a sessão
session_start();

// Verificar configuração inicial
if (!file_exists(DATA_PATH)) {
    mkdir(DATA_PATH, 0755, true);
}

// Verificar arquivos necessários
$arquivosRequeridos = [
    DATA_PATH . '/agendamentos.json',
    DATA_PATH . '/clientes.json',
    DATA_PATH . '/servicos.json'
];

$configOk = true;

foreach ($arquivosRequeridos as $arquivo) {
    if (!file_exists($arquivo)) {
        if (!touch($arquivo)) {
            $configOk = false;
        } else {
            // Inicializa com um array vazio em JSON
            file_put_contents($arquivo, json_encode([]));
        }
    }
    
    if (!is_writable($arquivo)) {
        $configOk = false;
    }
}

// Redirecionamento para configuração se necessário
if (!$configOk && basename($_SERVER['PHP_SELF']) !== 'configurar.php') {
    header('Location: ' . APP_URL . '/public/configurar.php');
    exit;
}

// Inicializar serviços
$agendamentoRepository = new \App\Repositories\AgendamentoRepository();
$clienteRepository = new \App\Repositories\ClienteRepository();
$servicoRepository = new \App\Repositories\ServicoRepository();
$funcionarioRepository = new \App\Repositories\FuncionarioRepository();

$idGenerator = new \App\Services\IdGeneratorService();
$dateHelper = new \App\Utils\DateTimeHelper();
$mailer = new \App\Utils\Mailer(
    MAIL_HOST, 
    MAIL_PORT, 
    MAIL_USERNAME, 
    MAIL_PASSWORD,
    MAIL_FROM,
    MAIL_FROM_NAME
);

// Inicializar serviços da aplicação
$clienteService = new \App\Services\ClienteService($clienteRepository, $idGenerator);
$servicoService = new \App\Services\ServicoService($servicoRepository);
$notificacaoService = new \App\Services\NotificacaoService($mailer);
$agendamentoService = new \App\Services\AgendamentoService(
    $agendamentoRepository, 
    $clienteRepository,
    $servicoRepository,
    $funcionarioRepository,
    $notificacaoService, 
    $dateHelper,
    $idGenerator
);

// Registrar manipuladores de comandos
$commandHandlers = [
    \App\Commands\CriarAgendamentoCommand::class => new \App\Handlers\CriarAgendamentoHandler($agendamentoService),
    \App\Commands\AtualizarAgendamentoCommand::class => new \App\Handlers\AtualizarAgendamentoHandler($agendamentoService),
    \App\Commands\CancelarAgendamentoCommand::class => new \App\Handlers\CancelarAgendamentoHandler($agendamentoService),
    \App\Commands\EnviarNotificacaoCommand::class => new \App\Handlers\EnviarNotificacaoHandler($notificacaoService)
];

// Função para executar comandos
function executeCommand($command) {
    global $commandHandlers;
    $handlerClass = get_class($command);
    if (isset($commandHandlers[$handlerClass])) {
        return $commandHandlers[$handlerClass]->handle($command);
    }
    throw new \Exception("Nenhum manipulador encontrado para " . $handlerClass);
}
```

### 3. Página Principal (`public/index.php`)

```php
<?php
require_once __DIR__ . '/../bootstrap.php';

// Verificar se o usuário está logado (para área administrativa)
$isAdmin = isset($_SESSION['admin']) && $_SESSION['admin'] === true;

// Data atual para exibição do calendário
$dataAtual = isset($_GET['data']) ? new DateTime($_GET['data']) : new DateTime();
$mes = $dataAtual->format('m');
$ano = $dataAtual->format('Y');

// Buscar agendamentos do dia (se for admin) ou agendamentos do usuário logado
if ($isAdmin) {
    $agendamentosHoje = $agendamentoService->buscarAgendamentosPorData($dataAtual->format('Y-m-d'));
} else {
    // Para clientes normais, mostrar apenas seus próprios agendamentos
    $clienteId = $_SESSION['cliente_id'] ?? null;
    $agendamentosHoje = $clienteId ? $agendamentoService->buscarAgendamentosPorCliente($clienteId) : [];
}

// Buscar serviços disponíveis
$servicos = $servicoService->listarServicos();

// Título da página
$pageTitle = APP_NAME . ' - ' . ($isAdmin ? 'Administração' : 'Agendamento');

// Incluir o cabeçalho
include TEMPLATES_PATH . '/header.php';
?>

<div class="container mt-4">
    <div class="row">
        <div class="col-lg-8">
            <div class="card shadow-sm">
                <div class="card-header bg-primary text-white">
                    <h4 class="mb-0">
                        <i class="fas fa-calendar-alt"></i> 
                        Agendamentos - <?= $dataAtual->format('d/m/Y') ?>
                    </h4>
                </div>
                <div class="card-body">
                    <!-- Navegação do calendário -->
                    <div class="d-flex justify-content-between align-items-center mb-4">
                        <a href="?data=<?= (clone $dataAtual)->modify('-1 day')->format('Y-m-d') ?>" class="btn btn-sm btn-outline-primary">
                            <i class="fas fa-chevron-left"></i> Dia anterior
                        </a>
                        <h5 class="mb-0"><?= $dataAtual->format('d/m/Y') ?></h5>
                        <a href="?data=<?= (clone $dataAtual)->modify('+1 day')->format('Y-m-d') ?>" class="btn btn-sm btn-outline-primary">
                            Próximo dia <i class="fas fa-chevron-right"></i>
                        </a>
                    </div>
                    
                    <!-- Horários e agendamentos -->
                    <div class="timeline">
                        <?php 
                        // Verificar dia da semana
                        $diaSemana = $dataAtual->format('w');
                        $horariosFuncionamento = $HORARIOS_FUNCIONAMENTO[$diaSemana] ?? null;
                        
                        if (!$horariosFuncionamento): 
                        ?>
                            <div class="alert alert-warning">
                                <i class="fas fa-exclamation-triangle"></i>
                                Não há atendimento nesta data.
                            </div>
                        <?php else: 
                            // Horário de abertura e fechamento
                            list($horaAbertura, $horaFechamento) = $horariosFuncionamento;
                            $horaInicio = new DateTime($dataAtual->format('Y-m-d') . ' ' . $horaAbertura);
                            $horaFim = new DateTime($dataAtual->format('Y-m-d') . ' ' . $horaFechamento);
                            $intervalo = new DateInterval('PT30M'); // 30 minutos
                            
                            // Percorrer os horários
                            $currentTime = clone $horaInicio;
                            while ($currentTime < $horaFim): 
                                $timeSlot = $currentTime->format('H:i');
                                $agendamentosNoHorario = array_filter($agendamentosHoje, function($agendamento) use ($timeSlot) {
                                    return $agendamento['hora'] === $timeSlot;
                                });
                        ?>
                            <div class="timeline-item">
                                <div class="timeline-time"><?= $timeSlot ?></div>
                                <div class="timeline-content">
                                    <?php if (empty($agendamentosNoHorario)): ?>
                                        <div class="timeline-slot available">
                                            <a href="agendamento.php?data=<?= $dataAtual->format('Y-m-d') ?>&horario=<?= $timeSlot ?>" class="btn btn-sm btn-outline-success">
                                                <i class="fas fa-plus"></i> Agendar
                                            </a>
                                        </div>
                                    <?php else: 
                                        foreach ($agendamentosNoHorario as $agendamento): 
                                            $status = $agendamento['status'];
                                            $statusInfo = $STATUS_AGENDAMENTO[$status];
                                    ?>
                                        <div class="timeline-slot booked bg-<?= $statusInfo['cor'] ?>-light">
                                            <div class="d-flex justify-content-between">
                                                <div>
                                                    <i class="<?= $statusInfo['icone'] ?>"></i>
                                                    <strong><?= $agendamento['cliente_nome'] ?></strong> - 
                                                    <?= $agendamento['servico_nome'] ?>
                                                </div>
                                                <?php if ($isAdmin): ?>
                                                    <div class="btn-group btn-group-sm">
                                                        <a href="atualizar_status.php?id=<?= $agendamento['id'] ?>&status=em_andamento" class="btn btn-info btn-sm">
                                                            <i class="fas fa-play"></i>
                                                        </a>
                                                        <a href="atualizar_status.php?id=<?= $agendamento['id'] ?>&status=concluido" class="btn btn-success btn-sm">
                                                            <i class="fas fa-check"></i>
                                                        </a>
                                                        <a href="atualizar_status.php?id=<?= $agendamento['id'] ?>&status=cancelado" class="btn btn-danger btn-sm">
                                                            <i class="fas fa-times"></i>
                                                        </a>
                                                    </div>
                                                <?php endif; ?>
                                            </div>
                                        </div>
                                    <?php 
                                        endforeach;
                                    endif; 
                                    ?>
                                </div>
                            </div>
                        <?php 
                                $currentTime->add($intervalo);
                            endwhile;
                        endif; 
                        ?>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-lg-4">
            <div class="card shadow-sm mb-4">
                <div class="card-header bg-success text-white">
                    <h4 class="mb-0"><i class="fas fa-car"></i> Nossos Serviços</h4>
                </div>
                <div class="card-body">
                    <div class="list-group">
                        <?php foreach ($servicos as $servico): ?>
                            <div class="list-group-item">
                                <div class="d-flex justify-content-between align-items-center">
                                    <h5 class="mb-1"><?= $servico['nome'] ?></h5>
                                    <span class="badge bg-primary rounded-pill">R$ <?= number_format($servico['preco'], 2, ',', '.') ?></span>
                                </div>
                                <p class="mb-1"><?= $servico['descricao'] ?></p>
                                <small class="text-muted">Duração: <?= $servico['duracao'] ?> minutos</small>
                            </div>
                        <?php endforeach; ?>
                    </div>
                </div>
            </div>
            
            <?php if (!isset($_SESSION['usuario'])): ?>
            <div class="card shadow-sm">
                <div class="card-header bg-info text-white">
                    <h4 class="mb-0"><i class="fas fa-sign-in-alt"></i> Login / Cadastro</h4>
                </div>
                <div class="card-body">
                    <p>Faça login ou cadastre-se para agendar um serviço.</p>
                    <div class="d-grid gap-2">
                        <a href="login.php" class="btn btn-primary">Login</a>
                        <a href="cadastro.php" class="btn btn-outline-secondary">Cadastre-se</a>
                    </div>
                </div>
            </div>
            <?php endif; ?>
        </div>
    </div>
</div>

<?php
// Incluir o rodapé
include TEMPLATES_PATH . '/footer.php';
?>
```

### 4. Formulário de Agendamento (`public/agendamento.php`)

```php
<?php
require_once __DIR__ . '/../bootstrap.php';

// Verificar se o usuário está logado
if (!isset($_SESSION['cliente_id'])) {
    // Salvar a URL atual para redirecionar após o login
    $_SESSION['redirect_after_login'] = $_SERVER['REQUEST_URI'];
    header('Location: login.php');
    exit;
}

$clienteId = $_SESSION['cliente_id'];
$cliente = $clienteService->buscarClientePorId($clienteId);

// Obter data e horário da URL
$data = isset($_GET['data']) ? $_GET['data'] : date('Y-m-d');
$horario = isset($_GET['horario']) ? $_GET['horario'] : '08:00';

// Validar data (não permitir agendamentos no passado)
$dataAgendamento = new DateTime($data);
$hoje = new DateTime();
if ($dataAgendamento < $hoje && $dataAgendamento->format('Y-m-d') !== $hoje->format('Y-m-d')) {
    // Redireciona para a data atual
    header('Location: agendamento.php?data=' . $hoje->format('Y-m-d') . '&horario=' . $horario);
    exit;
}

// Validar disponibilidade do horário
$disponivel = $agendamentoService->verificarDisponibilidade($data, $horario);
if (!$disponivel) {
    $_SESSION['erro'] = "O horário selecionado não está mais disponível.";
    header('Location: index.php?data=' . $data);
    exit;
}

// Buscar serviços disponíveis
$servicos = $servicoService->listarServicos();

// Processar o formulário de agendamento
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    try {
        $servicoId = $_POST['servico_id'];
        $observacoes = $_POST['observacoes'] ?? '';
        $veiculo = [
            'modelo' => $_POST['veiculo_modelo'],
            'placa' => $_POST['veiculo_placa'],
            'cor' => $_POST['veiculo_cor']
        ];
        
        // Criar comando para o agendamento
        $command = new \App\Commands\CriarAgendamentoCommand(
            $clienteId,
            $servicoId,
            $data,
            $horario,
            $veiculo,
            $observacoes
        );
        
        // Executar o comando
        $resultado = executeCommand($command);
        
        if ($resultado) {
            $_SESSION['sucesso'] = "Agendamento realizado com sucesso!";
            header('Location: meus_agendamentos.php');
            exit;
        }
        
    } catch (\Exception $e) {
        $_SESSION['erro'] = "Erro ao realizar agendamento: " . $e->getMessage();
    }
}

// Título da página
$pageTitle = APP_NAME . ' - Novo Agendamento';

// Incluir o cabeçalho
include TEMPLATES_PATH . '/header.php';
?>

<div class="container mt-4">
    <div class="card shadow">
        <div class="card-header bg-primary text-white">
            <h3><i class="fas fa-calendar-plus"></i> Novo Agendamento</h3>
        </div>
        <div class="card-body">
            <?php if (isset($_SESSION['erro'])): ?>
                <div class="alert alert-danger">
                    <?= $_SESSION['erro'] ?>
                    <?php unset($_SESSION['erro']); ?>
                </div>
            <?php endif; ?>
            
            <form method="post" action="">
                <div class="row mb-3">
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-header bg-light">
                                <h5 class="mb-0"><i class="fas fa-user"></i> Informações do Cliente</h5>
                            </div>
                            <div class="card-body">
                                <div class="mb-3">
                                    <label class="form-label">Nome:</label>
                                    <input type="text" class="form-control" value="<?= $cliente['nome'] ?>" readonly>
                                </div>
                                <div class="mb-3">
                                    <label class="form-label">Telefone:</label>
                                    <input type="text" class="form-control" value="<?= $cliente['telefone'] ?>" readonly>
                                </div>
                                <div class="mb-3">
                                    <label class="form-label">Email:</label>
                                    <input type="email" class="form-control" value="<?= $cliente['email'] ?>" readonly>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-header bg-light">
                                <h5 class="mb-0"><i class="fas fa-car"></i> Informações do Veículo</h5>
                            </div>
                            <div class="card-body">
                                <div class="mb-3">
                                    <label for="veiculo_modelo" class="form-label">Modelo:</label>
                                    <input type="text" class="form-control" id="veiculo_modelo" name="veiculo_modelo" required>
                                </div>
                                <div class="mb-3">
                                    <label for="veiculo_placa" class="form-label">Placa:</label>
                                    <input type="text" class="form-control" id="veiculo_placa" name="veiculo_placa" required>
                                </div>
                                <div class="mb-3">
                                    <label for="veiculo_cor" class="form-label">Cor:</label>
                                    <input type="text" class="form-control" id="veiculo_cor" name="veiculo_cor" required>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="row mb-3">
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-header bg-light">
                                <h5 class="mb-0"><i class="fas fa-calendar"></i> Data e Hora</h5>
                            </div>
                            <div class="card-body">
                                <div class="mb-3">
                                    <label class="form-label">Data:</label>
                                    <input type="date" class="form-control" value="<?= $data ?>" readonly>
                                </div>
                                <div class="mb-3">
                                    <label class="form-label">Horário:</label>
                                    <input type="text" class="form-control" value="<?= $horario ?>" readonly>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-header bg-light">
                                <h5 class="mb-0"><i class="fas fa-list"></i> Serviço</h5>
                            </div>
                            <div class="card-body">
                                <div class="mb-3">
                                    <label for="servico_id" class="form-label">Selecione o Serviço:</label>
                                    <select class="form-select" id="servico_id" name="servico_id" required>
                                        <option value="">Selecione...</option>
                                        <?php foreach ($servicos as $servico): ?>
                                            <option value="<?= $servico['id'] ?>" data-preco="<?= $servico['preco'] ?>" data-duracao="<?= $servico['duracao'] ?>">
                                                <?= $servico['nome'] ?> - R$ <?= number_format($servico['preco'], 2, ',', '.') ?>
                                            </option>
                                        <?php endforeach; ?>
                                    </select>
                                </div>
                                <div class="mb-3">
                                    <label for="observacoes" class="form-label">Observações:</label>
                                    <textarea class="form-control" id="observacoes" name="observacoes" rows="3"></textarea>
                                </div>
                                <div id="servicoInfo" class="alert alert-info d-none">
                                    <div><strong>Preço:</strong> <span id="precoServico">-</span></div>
                                    <div><strong>Duração:</strong> <span id="duracaoServico">-</span> minutos</div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="d-grid gap-2 d-md-flex justify-content-md-end">
                    <a href="index.php?data=<?= $data ?>" class="btn btn-secondary me-md-2">
                        <i class="fas fa-arrow-left"></i> Voltar
                    </a>
                    <button type="submit" class
