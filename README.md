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
