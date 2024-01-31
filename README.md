/**Esse é projeto é do módulo 11 e vai ensinar useEffect mais a fundo, ele consiste
 * em um site com vários cards que podem ser selecionados e vão para a parte superior do
 * site e lá, se clicado novamente, são excluídos. O site já veio baixado e
 * funcionando.
 */
/**Side effects são "tasks" que não impactam o ciclo de renderização corrente.
 * Assim, com a intenção de explicar na prática, foi criado na aula 176, o código abaixo,
 * a primeira linha é um código padrão para acessar a localização do usuário
 * e já é um código aceito pelos navegadores, o restante vai criar uma função
 * para arrumar os cards dos locais do site de acordo com a localização do usuário.
 * o sortPlacesByDistance é uma função que está localizada no componente loc.js e
 * com base na latitude e longitude cadastrada em cada item e lat e long obtidas
 * do usuário, vai calcular a distância de cada item, para isso foram passados
 * os argumentos na sequência1   
 */

navigator.geolocation.getCurrentPosition((position)=> {
    const sortedPlaces = sortPlacesByDistance(
       AVAILABLE_PLACES,
       position.coords.latitude,
       position.coords.longitude
     );
   }); 
/**na próxima aula foi criado um useState para guardar o array dos cards
 * já organizados por distância
 */
const [availablePlaces, setAvailablePlaces] = useState([]);
/**O código foi atualizado para ficar dentro de um useEffect e evitar um loop
 * infinito pois o hook garante que o código só rodará após a execução dos outros
 * componentes. O [] vazio será explicado depois
  */

useEffect(() => {
  navigator.geolocation.getCurrentPosition((position)=> {
   const sortedPlaces = sortPlacesByDistance(
      AVAILABLE_PLACES,
      position.coords.latitude,
      position.coords.longitude
    );
    setAvailablePlaces(sortedPlaces);
   });
  }, []); 

  /**o próximo código é um exemplo de que não se deve usar
   * o useEffect em todo efeito colateral, somente nos que renderizam junto
   * com o App, pois é um processo a mais para o browser rodar. O código
   * a seguir vai servir para guardar a informação do array de cards escolhidos
   * no storage do navegador e assim, não apagar ao se reiniciar a página. Ele
   * está dentro do function App, o que não seria possível se usássemos o useEffect
   * 
   */
  /** a variável storeIds vai guardar os locais selecionados pelo usuário
   * JSON.parse vai criar um objeto JS com base nos campos encontrados no array ou
   * || formar um array vazio, o if vai procurar um objeto com aquele id, o -1 
   * representa uma negativa, ou seja, se o objeto já não estiver no array, ele
   * vai ser incluído pelo setItem, que leva dois parâmetros, o array de cards
   * guardados e o um array com os id's, já em formato de string transformado 
   * pelo método. Vale ressaltar que esse código está dentro do handleSelectedPlace
   * e assim, só vai ser executado se o botão for clicado, mais um motivo
   * para não usar o useEffect  
   */
  const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
    if (storedIds.indexOf(id) === -1){
    localStorage.setItem(
    'selectedPlaces',
    JSON.stringify([id, ...storedIds])
   );
  }
}
/**Na aula 180, primeiro vai ser adicionado o código para deletar os itens
 * do local storage, ele vai ser incluído dentro da função handleRemovePlace
 * que cuida da exclusão do item também do card. A ideia aqui é que, uma vez
 * obtidos os itens (getItem) será setado (atualizado) o array do localstorage
 * através do método filter, esse método cria um novo array com base no anterior
 * e no critério estabelecido, nesse caso, se o id do card do local
 * selecionado for diferente, não deleta, se for igual, deleta...não entendi direito
 * mas funciona...creio...
 */

const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
    localStorage.setItem('selectedPlaces',
    JSON.stringify(storeIds.filter((id)=> id!== selectedPlace.current))
    );

/**Ainda na aula 180, é feito o código abaixo para conservar no localStorage
 * os itens selecionados e assim não se perder ao reiniciar a página. A ideia
 * é a mesma do array para guardar os itens selecionados, o código vai
 * pegar (get) os id's dos items do array "selectedPlace" ou um array vazio se 
 * nada for selecionado. Já a variável storedPlaces vai mapear o storedIds
 * e usar esses id's para encontrar (find) o item correspondente (place) no AVAILABLE_PLACES
 * que é o componente onde estão todos os itens. A ideia aqui foi demonstrar
 * que também não era necessário usar o useEffect para esse caso, pois a função
 * só tem que rodar uma vez ao se iniciar o código e depois ela é atualizada
 * automaticamente. Para garantir que só rodará uma vez, ele foi localizado
 * antes da função App. Mas, principalmente, porque o código roda
 * instantaneamente, ao contrário do geolocation, assim, o useEffect deve
 * ser usado para códigos que esperarm respostas de outros códigos,
 * API's normalmente.
 *  
 */

const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
const storedPlaces = storedIds.map((id)=> 
AVAILABLE_PLACES.find((place) => place.id === id)
);

/**As aulas 181 a 184 explicam que o segundo argumento do useEffect, o [], serve
 * para se passar quando o hook deve ser re-renderizado, ou seja, qual componente 
 * vai acionar o hook. No caso do código do curso foi passada o hook open, que está
 * vinculado com o useState abaixo
 */
 const [ modalIsOpen, setModalIsOpen] = useState(false);
  
 /**Já essa é a função q está no componente Modal.jsx */

function Modal({ open, children, onClose }) {
  const dialog = useRef();
 
  useEffect(() => {
    if (open) {
      dialog.current.showModal();
    } else {
      dialog.current.close();
    }
  }, [open]);
/**e abaixo é o componente modal no App.jsx */
  return (
<Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
  <DeleteConfirmation
    onCancel={handleStopRemovePlace}
    onConfirm={handleRemovePlace}
  />
</Modal>
/**Assim, a lógica aqui é a seguinte: o modal nesse caso é uma caixa
 * de confirmação que abre quando se clica no card e pergunta se confirma 
 * o delete. Antes ele era passado por useRef, agora, para fins de exemplo,
 * foi trocado para o useEffect. Portanto, o código do modal só vai ser chamado 
 * quando o prop open for acionado e este só vai ser chamado quando o useState
 * modalIsOpen for acionado e este, por sua vez, só vai ser acionado quando 
 * a função  function handleRemovePlace() for acionada, ou seja, quando
 * um item for clicado dentro dos locais selecionados. Também foi preciso
 * incluir o prop onClose pois ao se clicar o Esc a caixa de modal sumia
 * mas o estado não mudava (explicado na aula 184). O useEffect também garante
 * que o código só rode após o resto da página e assim não dê erro de não
 * referência caso algum item não tenha um estado inicial setado e seja
 * passado como referência. Effect dependencies são props, estados, funções ou
 * context values que dependam desses props ou estados que são usados dentro
 * da função que está dentro do useEffect./* 
 * 
 * Já na aula 186 é criado esse outro useEffect dentro do componente DeleteConfirmation
 * para que o código de deletar, que fica dentro da caixa de deletar,
 * espere 3 segundos ( 3000 milisegundos) e caso não seja clicado, ele exclui sozinho
 * o item. A novidade aqui é o return que é opcional, mas nesse caso garante que caso
 * se clique no deletar antes do 3 segundos, o setTimeout seja parado. O useEffect
 * nesse caso só foi chamado para poder parar o tempo, ou seja, ter um controle sobre
 * o que deve acontecer antes do código rodar novamente. 
 */ 
export default function DeleteConfirmation({ onConfirm, onCancel }) {
  useEffect (() => {
    const timer = setTimeout (() => {
      onConfirm();
    }, 3000);

    return () => {
      clearTimeout(timer);
    }
  }, [onConfirm]);
 
/**Na aula 187 foi incluido o props onConfirme como segundo argumento do useEffect
 * no entanto, foi explicado que poe acontecer um infinite loop a depender
 * do props ali passado, pois, nesse exemplo, há uma função controladora, mas
 * pode acontecer de se chamar só o props e quando for uma nova chamada ele
 * vai rodar novamente pois o React o lê como um objeto diferente a cada renderização
 * Para resolver isso será explicado o hook useCallback
 * O hook useCallback em React permite que você armazene em cache uma função e retorne
 *  uma versão memorizada dela. Isso pode ser usado para otimizar o desempenho
 *  em aplicativos React evitando re-renderizações desnecessárias.
 *  O useCallback aceita como primeiro parâmetro uma função e retorna uma versão
 *  memorizada dela (em termos de sua localização de memória, não o cálculo feito
 *  dentro). Isso significa que a função retornada não é recriada em uma nova 
 * referência de memória toda vez que o componente é re-renderizado, enquanto uma 
 * função normal dentro de um componente é.
 */

/**No exemplo da aula, a função para remover o card foi
 * incluída dentro de um useCallback que foi guardada dentro de 
 * uma variável de mesmo nome, assim, temos uma segurança extra de 
 * que não haverá re-renderização. Em relação ao segundo argumento, [], 
 * é a mesma lógica do useEffect, ali deve ir qualquer props ou função
 * que esteja dentro da função e possa alterar o seu estado,
 * no presente caso, não há nenhum, por isso, ficou em branco.
 */

const handleRemovePlace = useCallback( function handleRemovePlace() {
  setPickedPlaces((prevPickedPlaces) =>
    prevPickedPlaces.filter((place) => place.id !== selectedPlace.current)
  );
  setModalIsOpen(false);

  const storedIds = JSON.parse(localStorage.getItem('selectedPlaces')) || [];
  localStorage.setItem('selectedPlaces',
  JSON.stringify(storedIds.filter((id)=> id !== selectedPlace.current))
  );
}, []);

/**Na aula 189 foi criada uma função para mostrar ao usuário que o modal
 * de confirmação da exclusão do card vai durar 3 segundos, para isso vai ser acrescentada
 *uma barra de progresso que vai diminuindo durante 3000 mili segundos.
 Primeiro, foi incluída a seguinte tag no return do componente Delete Confirmation
 */
<progress value={remainingTime} max={TIMER} />

/**Então foi criada a função abaixo, uma variável global para guardar o tempo,
 * um useState para controle de estado desse tempo, um useEffect para evitar
 * o loop infinito, o setInterval que é uma função nativa do JS e que executa
 * determinada função repetidamente no tempo determinado, no caso, a cada dez milisegundos
 * , então a função para alterar o estado setRemainingTime pega o estado anterior
 * menos 10, já o return, vai chamar a também função interna clearInterval
 * para limpar o intervalo quando ele não for mais necessário, ou seja, quando
 * o card fechar
 * 
 */
const TIMER = 3000;

const [remainingTime, setRemainingTime] = useState(TIMER);

useEffect (() => {
  const interval = setInterval(() => {
    setRemainingTime(prevTime => prevTime - 10);
  }, 10);
  return () => {
    clearInterval(interval)
  };
}, []);

/**Na aula 190, a última do módulo, a função acima explicada, de controle de
 * tempo, foi inserida em um novo componente ProgressBar.jsx e somente esse componente
 * foi importado, assim, evita-se que todo o componente seja re-renderizado a cada
 * dez milisegundos, mas somente esse. Não vou copiar o componente todo, pois é basicamente
 * a mesma coisa, abaixo, só a importação dele com o devido prop
 */
 
 <ProgressBar timer={TIMER} />
