``` Typescript

export class Item {
  // Campo privado que guarda a descrição.
  constructor(private description: string) {}

  // Getter público para acessar a descrição (mantém o campo privado).
  getDescription() { return this.description; }

  // Permite atualizar a descrição da instância.
  updateDescription(newDescription: string) { this.description = newDescription; }

  // toJSON garante que ao serializar o objeto ele vire um POJO com a propriedade description.
  toJSON() { return { description: this.description }; }
}

export class ToDo {
  // Armazena os itens na memória (array síncrono), evitando usar Promise como estado.
  private items: Item[] = [];

  // O filepath onde os dados serão salvos/recuperados.
  constructor(private filepath: string) {
    // Inicializa carregando do arquivo sem bloquear o construtor.
    void this.init();
  }

  // Carrega o estado inicial do arquivo e popula this.items.
  private async init() {
    this.items = await this.loadFromFile();
  }

  // Serializa itens para JSON (usando toJSON de cada Item) e grava no arquivo.
  private async saveToFile() {
    const file = Bun.file(this.filepath);
    const data = JSON.stringify(this.items.map(i => i.toJSON()));
    return Bun.write(file, data);
  }

  // Lê o arquivo, faz parse do JSON e reconstrói instâncias de Item.
  private async loadFromFile(): Promise<Item[]> {
    const file = Bun.file(this.filepath);
    if (!(await file.exists())) return []; // retorna array vazio se não existir arquivo.
    const data = await file.text();
    const parsed = JSON.parse(data) as Array<{description:string}>;
    return parsed.map(p => new Item(p.description));
  }

  // Adiciona um item ao array em memória e espera a gravação no disco.
  async addItem(item: Item) {
    this.items.push(item);
    await this.saveToFile();
  }

  // Retorna uma cópia rasa do array de itens para evitar mutação externa.
  async getItems() { return this.items.slice(); }

  // Substitui o item no índice (valida bounds) e salva.
  async updateItem(index: number, newItem: Item) {
    if (index < 0 || index >= this.items.length) throw new Error('Index out of bounds');
    this.items[index] = newItem;
    await this.saveToFile();
  }

  // Remove o item no índice (valida bounds) e salva.
  async removeItem(index: number) {
    if (index < 0 || index >= this.items.length) throw new Error('Index out of bounds');
    this.items.splice(index, 1);
    await this.saveToFile();
  }

  // Procura por descrição usando o getter do Item (evita toJSON).
  async findItemByDescription(description: string) {
    return this.items.find(i => i.getDescription() === description);
  }

  // Retorna o item no índice após validar limites.
  async findItemByIndex(index: number) {
    if (index < 0 || index >= this.items.length) throw new Error('Index out of bounds');
    return this.items[index];
  }
}

```
