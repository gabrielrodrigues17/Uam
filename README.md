import java.util.ArrayList;
import java.util.List;

public class Usuario {
    private static int nextId = 1;
    
    private int id;
    private String nome;
    private String email;
    private String cidade;
    private List<Evento> eventosInscritos;
    
    public Usuario(String nome, String email, String cidade) {
        this.id = nextId++;
        this.nome = nome;
        this.email = email;
        this.cidade = cidade;
        this.eventosInscritos = new ArrayList<>();
    }
    
    // Getters e Setters
    public int getId() { return id; }
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getCidade() { return cidade; }
    public void setCidade(String cidade) { this.cidade = cidade; }
    public List<Evento> getEventosInscritos() { return eventosInscritos; }
    
    public void participarEvento(Evento evento) {
        if (!eventosInscritos.contains(evento)) {
            eventosInscritos.add(evento);
            evento.adicionarParticipante(this);
        }
    }
    
    public void cancelarParticipacao(Evento evento) {
        if (eventosInscritos.contains(evento)) {
            eventosInscritos.remove(evento);
            evento.removerParticipante(this);
        }
    }
    
    @Override
    public String toString() {
        return "Usuario [id=" + id + ", nome=" + nome + ", email=" + email + ", cidade=" + cidade + "]";
    }
}

import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;

public class Evento {
    private static int nextId = 1;
    
    private int id;
    private String nome;
    private String endereco;
    private String categoria;
    private LocalDateTime horario;
    private String descricao;
    private List<Usuario> participantes;
    
    public Evento(String nome, String endereco, String categoria, LocalDateTime horario, String descricao) {
        this.id = nextId++;
        this.nome = nome;
        this.endereco = endereco;
        this.categoria = categoria;
        this.horario = horario;
        this.descricao = descricao;
        this.participantes = new ArrayList<>();
    }
    
    // Getters e Setters
    public int getId() { return id; }
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public String getEndereco() { return endereco; }
    public void setEndereco(String endereco) { this.endereco = endereco; }
    public String getCategoria() { return categoria; }
    public void setCategoria(String categoria) { this.categoria = categoria; }
    public LocalDateTime getHorario() { return horario; }
    public void setHorario(LocalDateTime horario) { this.horario = horario; }
    public String getDescricao() { return descricao; }
    public void setDescricao(String descricao) { this.descricao = descricao; }
    public List<Usuario> getParticipantes() { return participantes; }
    
    public void adicionarParticipante(Usuario usuario) {
        if (!participantes.contains(usuario)) {
            participantes.add(usuario);
        }
    }
    
    public void removerParticipante(Usuario usuario) {
        participantes.remove(usuario);
    }
    
    public boolean isOcorrendoAgora() {
        LocalDateTime agora = LocalDateTime.now();
        return !agora.isBefore(horario) && agora.isBefore(horario.plusHours(2));
    }
    
    public boolean isFuturo() {
        return LocalDateTime.now().isBefore(horario);
    }
    
    public boolean isPassado() {
        return LocalDateTime.now().isAfter(horario.plusHours(2));
    }
    
    @Override
    public String toString() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
        return "Evento [id=" + id + ", nome=" + nome + ", endereco=" + endereco + 
               ", categoria=" + categoria + ", horario=" + horario.format(formatter) + 
               ", descricao=" + descricao + ", participantes=" + participantes.size() + "]";
    }
    
    public String toFileString() {
        return id + ";" + nome + ";" + endereco + ";" + categoria + ";" + 
               horario.toString() + ";" + descricao;
    }
    
    public static Evento fromFileString(String line) {
        String[] parts = line.split(";");
        if (parts.length < 6) return null;
        
        int id = Integer.parseInt(parts[0]);
        String nome = parts[1];
        String endereco = parts[2];
        String categoria = parts[3];
        LocalDateTime horario = LocalDateTime.parse(parts[4]);
        String descricao = parts[5];
        
        Evento evento = new Evento(nome, endereco, categoria, horario, descricao);
        evento.id = id;
        if (id >= nextId) nextId = id + 1;
        
        return evento;
    }
}
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.Scanner;

public class GerenciadorEventos {
    private List<Evento> eventos;
    private List<Usuario> usuarios;
    private Usuario usuarioLogado;
    private String arquivoEventos;
    private Scanner scanner;
    
    public GerenciadorEventos() {
        this.eventos = new ArrayList<>();
        this.usuarios = new ArrayList<>();
        this.arquivoEventos = "events.data";
        this.scanner = new Scanner(System.in);
        carregarEventos();
    }
    
    public void carregarEventos() {
        eventos.clear();
        try (BufferedReader reader = new BufferedReader(new FileReader(arquivoEventos))) {
            String line;
            while ((line = reader.readLine()) != null) {
                Evento evento = Evento.fromFileString(line);
                if (evento != null) {
                    eventos.add(evento);
                }
            }
            System.out.println("Eventos carregados com sucesso!");
        } catch (IOException e) {
            System.out.println("Arquivo de eventos não encontrado. Criando novo arquivo.");
        }
    }
    
    public void salvarEventos() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(arquivoEventos))) {
            for (Evento evento : eventos) {
                writer.write(evento.toFileString());
                writer.newLine();
            }
            System.out.println("Eventos salvos com sucesso!");
        } catch (IOException e) {
            System.out.println("Erro ao salvar eventos: " + e.getMessage());
        }
    }
    
    public void executar() {
        System.out.println("=== SISTEMA DE CADASTRO E NOTIFICAÇÃO DE EVENTOS ===");
        
        // Login ou cadastro de usuário
        if (usuarioLogado == null) {
            gerenciarUsuario();
        }
        
        int opcao;
        do {
            exibirMenu();
            opcao = scanner.nextInt();
            scanner.nextLine(); // Limpar buffer
            
            switch (opcao) {
                case 1:
                    cadastrarEvento();
                    break;
                case 2:
                    listarEventos();
                    break;
                case 3:
                    inscreverEmEvento();
                    break;
                case 4:
                    cancelarInscricao();
                    break;
                case 5:
                    listarMeusEventos();
                    break;
                case 6:
                    listarEventosPorStatus();
                    break;
                case 7:
                    System.out.println("Saindo do sistema...");
                    salvarEventos();
                    break;
                default:
                    System.out.println("Opção inválida!");
            }
        } while (opcao != 7);
    }
    
    private void exibirMenu() {
        System.out.println("\n===== MENU PRINCIPAL =====");
        System.out.println("Usuário: " + usuarioLogado.getNome());
        System.out.println("1. Cadastrar evento");
        System.out.println("2. Listar todos os eventos");
        System.out.println("3. Inscrever-se em evento");
        System.out.println("4. Cancelar inscrição em evento");
        System.out.println("5. Meus eventos");
        System.out.println("6. Eventos por status (ocorrendo agora, futuros, passados)");
        System.out.println("7. Sair");
        System.out.print("Escolha uma opção: ");
    }
    
    private void gerenciarUsuario() {
        System.out.println("\n===== GERENCIAMENTO DE USUÁRIO =====");
        System.out.println("1. Fazer login");
        System.out.println("2. Cadastrar novo usuário");
        System.out.print("Escolha uma opção: ");
        
        int opcao = scanner.nextInt();
        scanner.nextLine(); // Limpar buffer
        
        if (opcao == 1) {
            fazerLogin();
        } else if (opcao == 2) {
            cadastrarUsuario();
        } else {
            System.out.println("Opção inválida! Criando novo usuário.");
            cadastrarUsuario();
        }
    }
    
    private void fazerLogin() {
        System.out.print("Digite seu email: ");
        String email = scanner.nextLine();
        
        for (Usuario usuario : usuarios) {
            if (usuario.getEmail().equalsIgnoreCase(email)) {
                usuarioLogado = usuario;
                System.out.println("Login realizado com sucesso! Bem-vindo, " + usuario.getNome());
                return;
            }
        }
        
        System.out.println("Usuário não encontrado. Criando novo cadastro.");
        cadastrarUsuario();
    }
    
    private void cadastrarUsuario() {
        System.out.print("Digite seu nome: ");
        String nome = scanner.nextLine();
        
        System.out.print("Digite seu email: ");
        String email = scanner.nextLine();
        
        System.out.print("Digite sua cidade: ");
        String cidade = scanner.nextLine();
        
        Usuario novoUsuario = new Usuario(nome, email, cidade);
        usuarios.add(novoUsuario);
        usuarioLogado = novoUsuario;
        
        System.out.println("Usuário cadastrado com sucesso! Bem-vindo, " + nome);
    }
    
    private void cadastrarEvento() {
        System.out.println("\n===== CADASTRAR NOVO EVENTO =====");
        
        System.out.print("Nome do evento: ");
        String nome = scanner.nextLine();
        
        System.out.print("Endereço: ");
        String endereco = scanner.nextLine();
        
        System.out.print("Categoria (Festa, Esportivo, Show, Cultural, Educativo): ");
        String categoria = scanner.nextLine();
        
        LocalDateTime horario = null;
        while (horario == null) {
            System.out.print("Data e hora (formato: dd/MM/yyyy HH:mm): ");
            String dataHoraStr = scanner.nextLine();
            
            try {
                DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
                horario = LocalDateTime.parse(dataHoraStr, formatter);
                
                // Verificar se a data é futura
                if (horario.isBefore(LocalDateTime.now())) {
                    System.out.println("A data do evento deve ser futura!");
                    horario = null;
                }
            } catch (DateTimeParseException e) {
                System.out.println("Formato de data/hora inválido! Use o formato dd/MM/yyyy HH:mm");
            }
        }
        
        System.out.print("Descrição: ");
        String descricao = scanner.nextLine();
        
        Evento novoEvento = new Evento(nome, endereco, categoria, horario, descricao);
        eventos.add(novoEvento);
        
        System.out.println("Evento cadastrado com sucesso!");
    }
    
    private void listarEventos() {
        System.out.println("\n===== TODOS OS EVENTOS CADASTRADOS =====");
        
        if (eventos.isEmpty()) {
            System.out.println("Nenhum evento cadastrado.");
            return;
        }
        
        // Ordenar eventos por data (mais próximos primeiro)
        eventos.sort(Comparator.comparing(Evento::getHorario));
        
        for (Evento evento : eventos) {
            System.out.println(evento);
            
            if (evento.isOcorrendoAgora()) {
                System.out.println("  → ESTE EVENTO ESTÁ OCORRENDO AGORA!");
            } else if (evento.isFuturo()) {
                System.out.println("  → Evento futuro");
            } else {
                System.out.println("  → Evento já ocorrido");
            }
            
            System.out.println();
        }
    }
    
    private void inscreverEmEvento() {
        System.out.println("\n===== INSCREVER-SE EM EVENTO =====");
        
        if (eventos.isEmpty()) {
            System.out.println("Nenhum evento disponível para inscrição.");
            return;
        }
        
        // Listar apenas eventos futuros
        List<Evento> eventosFuturos = new ArrayList<>();
        for (Evento evento : eventos) {
            if (evento.isFuturo()) {
                eventosFuturos.add(evento);
            }
        }
        
        if (eventosFuturos.isEmpty()) {
            System.out.println("Nenhum evento futuro disponível para inscrição.");
            return;
        }
        
        for (int i = 0; i < eventosFuturos.size(); i++) {
            System.out.println((i + 1) + ". " + eventosFuturos.get(i).getNome() + 
                             " - " + eventosFuturos.get(i).getHorario().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")));
        }
        
        System.out.print("Escolha o número do evento para se inscrever: ");
        int escolha = scanner.nextInt();
        scanner.nextLine(); // Limpar buffer
        
        if (escolha < 1 || escolha > eventosFuturos.size()) {
            System.out.println("Escolha inválida!");
            return;
        }
        
        Evento eventoEscolhido = eventosFuturos.get(escolha - 1);
        usuarioLogado.participarEvento(eventoEscolhido);
        
        System.out.println("Inscrição realizada com sucesso no evento: " + eventoEscolhido.getNome());
    }
    
    private void cancelarInscricao() {
        System.out.println("\n===== CANCELAR INSCRIÇÃO =====");
        
        List<Evento> meusEventos = usuarioLogado.getEventosInscritos();
        
        if (meusEventos.isEmpty()) {
            System.out.println("Você não está inscrito em nenhum evento.");
            return;
        }
        
        for (int i = 0; i < meusEventos.size(); i++) {
            System.out.println((i + 1) + ". " + meusEventos.get(i).getNome() + 
                             " - " + meusEventos.get(i).getHorario().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")));
        }
        
        System.out.print("Escolha o número do evento para cancelar a inscrição: ");
        int escolha = scanner.nextInt();
        scanner.nextLine(); // Limpar buffer
        
        if (escolha < 1 || escolha > meusEventos.size()) {
            System.out.println("Escolha inválida!");
            return;
        }
        
        Evento eventoEscolhido = meusEventos.get(escolha - 1);
        usuarioLogado.cancelarParticipacao(eventoEscolhido);
        
        System.out.println("Inscrição cancelada com sucesso no evento: " + eventoEscolhido.getNome());
    }
    
    private void listarMeusEventos() {
        System.out.println("\n===== MEUS EVENTOS =====");
        
        List<Evento> meusEventos = usuarioLogado.getEventosInscritos();
        
        if (meusEventos.isEmpty()) {
            System.out.println("Você não está inscrito em nenhum evento.");
            return;
        }
        
        // Ordenar eventos por data
        meusEventos.sort(Comparator.comparing(Evento::getHorario));
        
        for (Evento evento : meusEventos) {
            System.out.println(evento);
            
            if (evento.isOcorrendoAgora()) {
                System.out.println("  → ESTE EVENTO ESTÁ OCORRENDO AGORA!");
            } else if (evento.isFuturo()) {
                System.out.println("  → Evento futuro");
            } else {
                System.out.println("  → Evento já ocorrido");
            }
            
            System.out.println();
        }
    }
    
    private void listarEventosPorStatus() {
        System.out.println("\n===== EVENTOS POR STATUS =====");
        System.out.println("1. Eventos ocorrendo agora");
        System.out.println("2. Eventos futuros");
        System.out.println("3. Eventos passados");
        System.out.print("Escolha uma opção: ");
        
        int opcao = scanner.nextInt();
        scanner.nextLine(); // Limpar buffer
        
        List<Evento> eventosFiltrados = new ArrayList<>();
        String status = "";
        
        switch (opcao) {
            case 1:
                for (Evento evento : eventos) {
                    if (evento.isOcorrendoAgora()) {
                        eventosFiltrados.add(evento);
                    }
                }
                status = "OCORRENDO AGORA";
                break;
            case 2:
                for (Evento evento : eventos) {
                    if (evento.isFuturo()) {
                        eventosFiltrados.add(evento);
                    }
                }
                status = "FUTUROS";
                break;
            case 3:
                for (Evento evento : eventos) {
                    if (evento.isPassado()) {
                        eventosFiltrados.add(evento);
                    }
                }
                status = "PASSADOS";
                break;
            default:
                System.out.println("Opção inválida!");
                return;
        }
        
        System.out.println("\n===== EVENTOS " + status + " =====");
        
        if (eventosFiltrados.isEmpty()) {
            System.out.println("Nenhum evento encontrado.");
            return;
        }
        
        // Ordenar eventos por data
        eventosFiltrados.sort(Comparator.comparing(Evento::getHorario));
        
        for (Evento evento : eventosFiltrados) {
            System.out.println(evento);
            System.out.println();
        }
    }
    
    public static void main(String[] args) {
        GerenciadorEventos sistema = new GerenciadorEventos();
        sistema.executar();
    }
}
