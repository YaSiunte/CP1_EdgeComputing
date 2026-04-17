# Projeto InVino - Sistema de Monitoramento de Luminosidade

O InVino é um sistema desenvolvido em um Arduino para o controle e monitoramento de luz em ambientes controlados. O projeto utiliza um sensor de luminosidade (LDR) e fornece alertas visuais e sonoros baseados em faixas de segurança pré-definidas.

-----

## Descrição do Projeto

O sistema realiza a leitura analógica do ambiente e processa os dados utilizando uma média móvel de 10 amostras para evitar leituras instáveis. O feedback e fornecido através de um display LCD, três LEDs coloridos e um buzzer.

O funcionamento segue a seguinte lógica de estados:
1. OK: Luminosidade entre 30% e 70%. LED Verde aceso.
2. ALERTA: Luminosidade entre 15-29% ou 71-85%. LED Amarelo aceso e Buzzer acionado por 3 segundos.
3. PROBLEMA: Luminosidade abaixo de 15% ou acima de 85%. LED Vermelho aceso.

-----

## Hardware e Conexões

Abaixo estão os pinos utilizados no Arduino:

- LCD RS: Pino 12
- LCD Enable: Pino 11
- LCD D4-D7: Pinos 5, 4, 3, 2
- LED Verde: Pino 6
- LED Amarelo: Pino 7
- LED Vermelho: Pino 8
- Buzzer: Pino 9
- Sensor LDR: Pino A0 (Analógico)

-----

## Dependências

- Biblioteca: LiquidCrystal.h (Já instalada por padrão na Arduino IDE)

-----

## Como Utilizar

1. Conecte os componentes conforme a lista de hardware acima.
2. Certifique-se de que o sensor LDR esteja configurado com um resistor de 10k ohms em modo divisor de tensão.
3. Abra o código fonte na Arduino IDE.
4. Conecte o Arduino ao computador e selecione a porta COM correspondente.
5. Realize o Upload do codigo.
6. Acompanhe os dados pelo Display LCD ou pelo Monitor Serial.

-----

## Detalhes Técnicos

- Lógica de Média Móvel: O sistema calcula a media das últimas 10 leituras para suavizar a transição entre os estados.
- Gerenciamento do Buzzer: No estado de Alerta, o buzzer e ativado de forma não bloqueante (utilizando a função millis), permitindo que o sistema continue lendo o sensor enquanto o som e emitido.
- Interface Customizada: O sistema carrega caracteres especiais no setup para exibir o logotipo InVino no LCD durante a inicialização.
