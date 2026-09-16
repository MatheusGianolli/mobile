1. Consulta de dados remotos (0,5 ponto)
A finalidade de uma query no TanStack Query é buscar, armazenar em cache e gerenciar o ciclo de vida de dados assíncronos (geralmente provenientes de requisições GET para uma API). Ela abstrai a complexidade de lidar com requisições repetidas, garantindo que os dados estejam disponíveis rapidamente para a interface.

Uma interface precisa considerar os estados da consulta porque requisições de rede não são instantâneas e podem falhar:

Carregamento (isPending / isLoading): Fornece feedback visual ao usuário (como um spinner), indicando que a aplicação está processando a requisição. Sem isso, a tela pareceria travada.

Sucesso (isSuccess): Garante que a interface só tente renderizar as informações quando elas de fato chegarem e estiverem prontas.

Erro (isError): Permite tratar falhas de rede ou de servidor com elegância (exibindo mensagens de erro ou botões de tentar novamente), evitando que o aplicativo quebre silenciosamente.

2. Cache e atualização dos dados (1 ponto)
A queryKey atua como o identificador único de uma consulta no cache. É um array que o TanStack Query utiliza para rastrear, compartilhar, atualizar ou invalidar os dados referentes àquela requisição específica em toda a aplicação.

Diferenças entre os estados do cache:

Dado fresh (fresco): É um dado que acabou de ser buscado e ainda está dentro do tempo de validade definido pelo staleTime. Se um componente solicitar esse dado, ele será entregue do cache sem disparar uma nova requisição em segundo plano.

Dado stale (obsoleto): É um dado cujo staleTime expirou. O TanStack Query ainda o exibirá na tela imediatamente, mas disparará uma requisição em segundo plano para buscar possíveis atualizações (padrão stale-while-revalidate).

Consulta inativa: Ocorre quando um dado está no cache, mas não há nenhum componente montado na tela utilizando aquela queryKey no momento.

Remoção pelo gcTime (Garbage Collection Time): É o tempo que um dado inativo permanece na memória antes de ser excluído definitivamente. Se o componente for remontado antes desse tempo expirar, o dado inativo é reutilizado (e revalidado). Se o tempo estourar, o dado é deletado para liberar memória.

Um dado stale é necessariamente um dado incorreto?
Não. Ser stale significa apenas que o dado ultrapassou o tempo de garantia de frescor configurado no frontend, servindo como um "gatilho" para que o TanStack Query faça uma nova busca em segundo plano na próxima vez que a tela for acessada. O dado no servidor pode não ter mudado nada, o que significa que o dado stale em exibição continua perfeitamente correto.

3. Alterações e consistência (0,5 ponto)
Por que informações anteriores continuam aparecendo: O TanStack Query gerencia o estado assíncrono no frontend. Quando você faz uma mutation (POST, PUT, DELETE) e ela tem sucesso no servidor, o cache local não adivinha que os dados foram alterados. Ele continuará servindo os dados antigos até que algo o avise para buscar novamente.

Finalidade de invalidar uma consulta: Chamar a função invalidateQueries (passando a queryKey) sinaliza ao cache que aqueles dados específicos agora são inválidos. Se houver componentes ativos na tela usando esses dados, o TanStack Query fará um refetch automático e imediato, sincronizando a interface com o banco de dados.

Por que encapsular em um hook próprio: Criar um custom hook (ex: useCreateUser) centraliza a lógica da mutation e da invalidação do cache em um só lugar. Isso evita repetição de código, facilita a manutenção e mantém os componentes limpos, focados apenas em disparar a ação, sem precisar conhecer a estrutura do cache.

4. Autenticação, sessão e JWT (0,5 ponto)
Autenticar um usuário: É o ato de provar a identidade do usuário (por exemplo, enviando e-mail e senha para o servidor verificar se estão corretos).

Receber um token JWT: É a consequência de uma autenticação bem-sucedida. O servidor gera e devolve um "crachá" (o token) assinado digitalmente, que contém as credenciais de acesso temporário do usuário.

Manter uma sessão no aplicativo: É o processo de armazenar o JWT no frontend (seja em cookies ou local storage) e gerenciar seu ciclo de vida (quando ele expira, quando precisa ser renovado), para que o usuário não precise digitar a senha a cada clique.

Por que possuir um token e não usá-lo não mantém o usuário autenticado na API?
Porque as APIs modernas (como as baseadas em REST) são stateless (sem estado). O servidor não guarda na memória quem está logado. Para cada requisição em rotas protegidas, o aplicativo precisa explicitamente anexar o token (geralmente no cabeçalho Authorization: Bearer <token>). Se o token não for enviado, a API tratará a requisição como anônima e a rejeitará.

5. Estado de autenticação com Firebase (0,5 ponto)
Observar o estado com onAuthStateChanged é melhor porque a autenticação no Firebase é dinâmica. Se você checar o usuário apenas no momento do login, perderá o controle do estado se o usuário atualizar a página (F5) ou se o token expirar. O onAuthStateChanged funciona como um "ouvinte" contínuo.

Relação com o ecossistema da aplicação:

Restauração da sessão: Quando o usuário atualiza a página, o Firebase precisa de uma fração de segundo para ler os cookies/IndexedDB locais. O onAuthStateChanged dispara automaticamente assim que descobre o usuário salvo, restaurando a sessão sem pedir login de novo.

Telas protegidas: Como o ouvinte mantém uma "verdade única" e global sobre o status do usuário, você pode criar rotas que escutam esse estado e redirecionam o usuário instantaneamente para a tela de login caso ele não seja detectado.

Encerramento da sessão: Quando a função signOut() é chamada, você não precisa limpar estados manualmente por todo o aplicativo. O ouvinte detectará a mudança de estado para null e atualizará automaticamente o aplicativo inteiro, desativando telas protegidas.
