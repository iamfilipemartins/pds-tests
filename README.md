# Avaliando testes do Jest

Jest é um framework de teste em JavaScript projetado para garantir a correção de qualquer código JavaScript. Ele permite que você escreva testes com uma API acessível, familiar e rica em recursos que lhe dá resultados rapidamente.

Funciona em projetos que usam: Babel, TypeScript, Node, React, Angular, Vue e outros. 

## Teste 1 - Comando --silent que silencia os consoles durante os testes
O seguinte teste realiza a execução do jest, por meio da função runJest e recebe os resultados do teste por meio da função extractSummary, onde a principal função do teste é verificar a execução do Jest sem a realização de consoles durante as verificações. 

Esse comando de silenciar os consoles é feito frequentemente no ambiente profissional afim de facilitar a leitura dos resultados dos testes, tanto para desenvolvedores quanto nos scripts de CI.
 
As verificações do resultado desejado são feitas por meio de comparações usando a função expect, onde a mesma compara os valores recebidos no resultado do teste com o esperado. 

A função expect é usada toda vez que você quer testar um valor. Você raramente irá usar o expect sozinho. Em vez disso, você irá usar expect junto com uma função "matcher" para verificar algo sobre um valor. Neste caso e nos outros testes a seguir, o toBe e o toMatchSnapshot são as funções "matcher".
```typescript
test('does not print to console with --silent', () => {
  const {stderr, stdout, exitCode} = runJest('console', [
    // Need to pass --config because console test specifies `verbose: false`
    '--config=' +
      JSON.stringify({
        testEnvironment: 'node',
      }),
    '--silent',
    '--no-cache',
  ]);
  const {summary, rest} = extractSummary(stderr);

  expect(exitCode).toBe(0);
  expect(wrap(stdout)).toMatchSnapshot();
  expect(wrap(rest)).toMatchSnapshot();
  expect(wrap(summary)).toMatchSnapshot();
});
```
 
## Teste 2 - Verificando função .only para realizar testes específicos
O seguinte teste realiza a execução do jest, por meio da função runJest e recebe os resultados do teste por meio da função extractSummary, onde a principal função do teste é verificar a execução do Jest apenas em testes com a tag .only, onde a mesma especifica qual teste deve ser executado em determinado arquivo. 

Essa função para executar um teste específico também é utilizada frequentemente no ambiente profissional afim de facilitar na execução de um teste específico e obter o resultado do mesmo, seja porque o teste especificado estava com algum erro ou o fluxo testado no mesmo foi o único alterado durante a tarefa realizada.
 
As verificações do resultado desejado são feitas por meio de comparações usando a função expect, onde a mesma compara os valores recebidos no resultado do teste com o esperado.
```typescript
test('shows only the tests with .only as being ran', () => {
  const result = runJest(dir, ['eachOnly.test.js']);
  expect(result.exitCode).toBe(0);
  const {rest} = extractSummary(result.stderr);
  expect(wrap(rest)).toMatchSnapshot();
});
```
## Teste 3 - Verificação de timeouts nas requisições durante os testes
O seguinte teste realiza a execução do jest, por meio da função runJest e recebe os resultados do teste por meio da função extractSummary, onde a principal função do teste é verificar a execução do Jest dentro de um time específico definido, que é o máximo esperado para a execução do teste. 

Essa verificação é importante afim de controlar e analisar os tempos de execução de requisições realizadas dentro do fluxo, assim auxiliando no entendimento da efetividade do código desenvolvido.
 
As verificações do resultado desejado são feitas por meio de comparações usando a função expect, onde a mesma compara os valores recebidos no resultado do teste com o esperado.
```typescript
test('exceeds the timeout', () => {
  writeFiles(DIR, {
    '__tests__/a-banana.js': `
      jest.setTimeout(20);
      test('banana', () => {
        return new Promise(resolve => {
          setTimeout(resolve, 100);
        });
      });
    `,
    'package.json': '{}',
  });

  const {stderr, exitCode} = runJest(DIR, ['-w=1', '--ci=false']);
  const {rest, summary} = extractSummary(stderr);
  expect(rest).toMatch(
    /(jest\.setTimeout|jasmine\.DEFAULT_TIMEOUT_INTERVAL|Exceeded timeout)/,
  );
  expect(wrap(summary)).toMatchSnapshot();
  expect(exitCode).toBe(1);
});
```
