1. Elementos Arquiteturais (Seção 2.1)
 - Código ->

    int MEM[16];            // memória de dados simulada
    int PC = 0;             // program counter (controla a instrução atual/próxima)
    byte IR = 0;            // instruction register (armazena o opcode binário)
    int ACC = 0;            // acumulador (armazena operandos e resultados)
    bool FLAG_Z = false;    // flag de comparação
    bool EXECUTANDO = true; // controle de execução do programa

 - Exige que todas as variaveis estajam com seus respectivos nomes e dados sketch indentificáveis, também exige que cada cotenha comentários explicando sua função.

2.  Estruturas de memória de programa e operação (Seção 2.2, 2.2.1)
 - Código ->
    
        struct Instrucao {
    byte opcode;
    int operando;
    };

    Instrucao memoriaPrograma[32]; // Armazena as instruções do usuário
    int ponteiroCarga = 0;         // Endereço para carregar nova instrução

    enum ModoOperacao { MODO_OCIOSO, MODO_LOAD, MODO_RUN };
    ModoOperacao modoAtual = MODO_OCIOSO;

    String bufferEntrada = "";
    bool aguardandoOperando = false;
    byte opcodeTemp = 0;

 - Exigiu 2 modelos distintos de funções: Load e Run. Suas instrições são armazenadas na memória, de forma que sua execução não seja automatica, utilizamos: *enum* *memoriaPrograma* *ponteiroCarga* pra fazer isso(modo Run). Também exige que uma memoria de instruções que opere de forma sequencial e ponteiro de carga, utilizamos: *memoriaPrograma* *ponteiroCarga* (modo Load).

3. Pinagem e Hardware (Seção 2.2.1, F01, F05, F06, F07, F08)
 - Código ->
   
      const int pinoA = 22, pinoB = 23, pinoC = 24, pinoD = 25;
   const int pinoE = 26, pinoF = 27, pinoG = 28;

   const byte LINHAS = 4;
   const byte COLUNAS = 4;
   char teclas[LINHAS][COLUNAS] = {
     {'1','2','3','A'}, // A = ENTER
     {'4','5','6','B'}, // B = RUN
     {'7','8','9','C'}, // C = CLEAR BUFFER
     {'*','0','#','D'}  // * = NEXT STEP, # = MODO LOAD
   };
   byte pinosLinhas[LINHAS] = {30, 31, 32, 33};
   byte pinosColunas[COLUNAS] = {34, 35, 36, 37};
   Keypad teclado = Keypad(makeKeymap(teclas), pinosLinhas, pinosColunas, LINHAS, COLUNAS);

   const int pinoTrig = 40;
   const int pinoEcho = 41;
   const int pinoLed1 = 42; // LED de Alerta principal
   const int pinoLed2 = 43;
   const int pinoLed3 = 44;
   const int pinoBuzzer = 45;

 - Exigiu um mecanismo de confirmação e de limpeza do buffer, foi definido Tecla *A* : Enter | Tecla *C* : Clear | Tecla *✳* : Próxima    Instrução | Tecla *#* : Modo Load. Possui entrada via teclado matricial com mapeamento de teclas.
   Exigiu entrada via teclado matricial com mapeamento de teclas. Matriz de teclas documentada com comentários para cada função.
   Tambem solicitou saídas de sensor, LED's, buzzer e display, estão todos conectados declarados como constantes e com seus respectivos nomes.

4. Tabela de Mnemonicos e ISA (Seção 2.2.1, Seção 2.3, F02)
 - Código ->

      String obterMnemonico(byte opcode) {
        switch(opcode) {
          case 0: return "NOP"; case 1: return "READ"; case 2: return "LOADK";
          case 3: return "ADDK"; case 4: return "SUBK"; case 5: return "CMPK";
          case 6: return "LEDON"; case 7: return "LEDOFF"; case 8: return "BUZON";
          case 9: return "BUZOFF"; case 10: return "DISP"; case 11: return "ALERT";
          case 12: return "BINC"; case 13: return "STORE"; case 14: return "LOADM";
          case 15: return "HALT"; default: return "???";
        }
      }

      bool precisaOperando(byte opcode) {
        //retorna true se a instrução exige um segunso número
        return (opcode == 2 || opcode == 3 || opcode == 4 || opcode == 5 || 
              opcode == 6 || opcode == 7 || opcode == 13 || opcode == 14);
      }

  - Exige uma rotina que associe mnemonicos a upcodes, armazenados numa IR. A função *obterMnemonico(byte opcode)* recebe o opcode e associa com o mnemonico. o O *precisaOperando(byte code)* garante que o sistema saiba exatamente quantos valores esperar para cada instrução antes de armazená-la. Exige que cada instrução tenha mnemônico, opcode de 4 bits e descrição funcional.

5. Setup Inicial(Seção 2.2, Seção 2.2.4, F05, F06, F07, F08 ) 
 - Código ->

    void setup() {
      Serial.begin(9600);
  
      pinMode(pinoA, OUTPUT); pinMode(pinoB, OUTPUT); pinMode(pinoC, OUTPUT);
      pinMode(pinoD, OUTPUT); pinMode(pinoE, OUTPUT); pinMode(pinoF, OUTPUT);
      pinMode(pinoG, OUTPUT); 
  
      pinMode(pinoLed1, OUTPUT); pinMode(pinoLed2, OUTPUT); pinMode(pinoLed3, OUTPUT);
      pinMode(pinoBuzzer, OUTPUT);
      pinMode(pinoTrig, OUTPUT); pinMode(pinoEcho, INPUT);

      limparDisplay();
  
      Serial.println("=========================================");
      Serial.println("  SISTEMA INTERPRETADOR INICIALIZADO     ");
      Serial.println("=========================================");
      Serial.println("Aperte '#' para entrar no MODO LOAD.");
      Serial.println("Aperte 'B' ou digite 'RUN' para MODO RUN.");
    }

 -    