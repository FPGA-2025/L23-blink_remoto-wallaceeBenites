# Começando do início  

Sempre que aprendemos uma nova linguagem de programação, escrevemos um "Hello World" para testar a linguagem e nos dar boa sorte. No mundo do hardware, esse "Hello World" geralmente se transforma em um "Blink", que consiste basicamente em fazer um LED piscar. Para manter a tradição, também iniciaremos com um Blink utilizando o kit FPGA.  

## Utilizando o Laboratório Remoto

Este laboratório utilizará a montagem remota com um FPGA sendo filmado por uma câmera. Você pode acessar o site com a filmagem no link [embarcatechfpga.lsc.ic.unicamp.br](https://embarcatechfpga.lsc.ic.unicamp.br). Foi feito um guia para usar o laboratório remoto, que está [aqui](guia_remoto.md).

## Construindo o Blink  

O Blink consiste, essencialmente, em fazer um LED piscar. Para isso, precisamos definir uma frequência de oscilação, o que requer um sinal de clock de referência. Neste circuito, utilizaremos um clock de 25 MHz fornecido por um pino da FPGA (esse clock vem de um cristal oscilador e tem o valor fixo). Dessa forma, nosso circuito terá como entrada um sinal de clock e um sinal de reset (ativo em nível lógico baixo) e, como saída, um LED.  

Internamente, usaremos um registrador de 32 bits para contar os ciclos de clock e determinar o momento de alternar o estado do LED. O circuito consiste basicamente em um somador que incrementa esse registrador a cada ciclo de clock. Quando ele atinge um valor predeterminado, o estado do LED é invertido.  

Por exemplo, para fazer o LED piscar uma vez a cada meio segundo com um clock de 25 MHz, precisamos contar até **12_500_000** ciclos. (Observe que utilizei **'_'** para separar as ordens de grandeza, em vez de **'.'**, pois essa notação pode ser usada em Verilog para representar números inteiros em diferentes bases, como decimal, binário, hexadecimal ou octal). Como nosso clock tem **25_000_000** ciclos por segundo contaremos até **12_500_000**, quando o contador atingir esse valor, inverteremos o estado do LED e reiniciaremos a contagem, desta forma inverteremos o estado do led a cada meio segundo.  

O código Verilog abaixo implementa esse circuito:  

```verilog
module Blink #(
    parameter CLK_FREQ = 25_000_000 
) (
    input wire clk,
    input wire rst_n,
    output reg [7:0] leds
);

localparam ONE_SECOND  = CLK_FREQ;
localparam HALF_SECOND = CLK_FREQ / 2;

reg [31:0] counter;

always @(posedge clk ) begin
    if (!rst_n) begin
        counter <= 32'h0;
        leds     <= 8'b0;
    end else begin
        if (counter >= HALF_SECOND - 1) begin
            counter <= 32'h0;
            leds[0]     <= ~leds[0];
        end else begin
            counter <= counter + 1;
        end
    end
end
    
endmodule
```    

## Simulando  

Antes de carregar um design em uma FPGA, uma boa prática é simulá-lo, pois depurar na placa é um processo muito mais difícil do que depurar na simulação.  

O código abaixo implementa um testbench básico para o Blink:  

```verilog
`timescale 1ns/1ns
module blink_tb();

reg clk, rst_n;
wire [7:0] leds;

initial begin
    $dumpfile("blink.vcd");
    $dumpvars;
    clk = 0;
    rst_n = 0;
    
    #2 rst_n = 1;
    #1000 $finish;
end

Blink #(
    .CLK_FREQ(50)
) U1(
    .clk   (clk),
    .rst_n (rst_n),
    .leds   (leds)
);

always #1 clk = ~clk;

endmodule
```

Esse testbench gera um arquivo **blink.vcd**, contendo a evolução dos sinais ao longo do tempo. Esse arquivo pode ser visualizado com ferramentas como **GTKWave** ou a extensão **WaveTrace** no VSCode. Note que o valor de `CLK_FREQ` foi reduzido para facilitar a simulação.

## Atividade

Para se familiarizar com o kit FPGA, faça as seguintes modificações no código do blink.v, uma de cada vez:

1. Modifique qual/quais led piscam para ver como o código é mapeado na placa.
2. Mude a frequência com que o led pisca fazendo ele piscar mais rápido ou mais devagar
3. Modifique o código para que o led fique muito tempo aceso e pouco tempo apagado

## Entrega  

O objetivo desta atividade é familiarizar-se com o ambiente de desenvolvimento. Como entrega, basta enviar o repositório com o arquivo **blink.v** para o GitHub.  

O **GitHub Classroom** já está configurado para enviar automaticamente o código para o laboratório remoto.  

> Para enviar para a correção automática, volte o código do blink.v para o código original fornecido neste README.

Para as próximas atividades, forneceremos apenas a especificação do módulo principal (top module), incluindo entradas, saídas, nomes de sinais e o arquivo e pinout para a placa FPGA. 

> **Dica**
    Os testes do GitHub estão embutidos nos arquivos do laboratório. Se quiser saber mais sobre eles, consulte o script de correção `run sh` no repositório do GitHub. **Não altere os arquivos de correção!**   