# Criando-um-Gerenciador-de-Cards-de-Pokémon-com-Flutter

Desenvolver um aplicativo de Gerenciador de Cards de Pokémon com Flutter envolve a integração de uma API para obter dados dos Pokémon, o uso do MobX para gerenciamento de estado reativo e a arquitetura Modular para organização do projeto, incluindo injeção de dependência e gerenciamento de rotas. Vamos abordar cada parte deste projeto de forma detalhada e organizada.

### Configuração Inicial do Projeto

1. **Setup do Projeto:**
   - Instale o Flutter em sua máquina conforme a documentação oficial.
   - Crie um novo projeto Flutter com o comando `flutter create nome_do_projeto`.

2. **Estrutura do Projeto:**
   Organize o projeto em módulos para facilitar a manutenção e expansão.
   ```
   /nome_do_projeto
   ├── /lib
   │   ├── /core
   │   │   ├── api.dart
   │   │   ├── models
   │   │   │   ├── pokemon.dart
   │   ├── /modules
   │   │   ├── /pokemon
   │   │   │   ├── pokemon_module.dart
   │   │   │   ├── pokemon_page.dart
   │   │   │   ├── pokemon_store.dart
   │   ├── main.dart
   ├── pubspec.yaml
   ```

### Integração com API de Pokémon

1. **Configuração da API:**
   - Utilize a PokeAPI (https://pokeapi.co/) para obter dados dos Pokémon.
   - Crie um arquivo `api.dart` para gerenciar as chamadas à API.

   Exemplo básico de configuração da API:
   ```dart
   // /lib/core/api.dart

   import 'dart:convert';
   import 'package:http/http.dart' as http;
   import 'package:nome_do_projeto/core/models/pokemon.dart';

   class ApiService {
     static const String baseUrl = 'https://pokeapi.co/api/v2';

     Future<List<Pokemon>> fetchPokemons() async {
       final response = await http.get(Uri.parse('$baseUrl/pokemon'));
       if (response.statusCode == 200) {
         final jsonData = jsonDecode(response.body);
         List<Pokemon> pokemons = [];
         for (var item in jsonData['results']) {
           Pokemon pokemon = Pokemon.fromJson(item);
           pokemons.add(pokemon);
         }
         return pokemons;
       } else {
         throw Exception('Falha ao carregar lista de Pokémon');
       }
     }
   }
   ```

   Exemplo de modelo (`pokemon.dart`):
   ```dart
   // /lib/core/models/pokemon.dart

   class Pokemon {
     final String name;
     final String url;

     Pokemon({required this.name, required this.url});

     factory Pokemon.fromJson(Map<String, dynamic> json) {
       return Pokemon(
         name: json['name'],
         url: json['url'],
       );
     }
   }
   ```

### Gerenciamento de Estado com MobX

1. **Configuração do MobX:**
   - Configure o pacote `mobx` no arquivo `pubspec.yaml`.
   - Implemente um store para gerenciar o estado dos Pokémon.

   Exemplo de store (`pokemon_store.dart`):
   ```dart
   // /lib/modules/pokemon/pokemon_store.dart

   import 'package:mobx/mobx.dart';
   import 'package:nome_do_projeto/core/models/pokemon.dart';
   import 'package:nome_do_projeto/core/api.dart';

   part 'pokemon_store.g.dart';

   class PokemonStore = _PokemonStoreBase with _$PokemonStore;

   abstract class _PokemonStoreBase with Store {
     final ApiService _api = ApiService();

     @observable
     ObservableList<Pokemon> pokemons = ObservableList<Pokemon>();

     @action
     Future<void> fetchPokemons() async {
       try {
         final List<Pokemon> fetchedPokemons = await _api.fetchPokemons();
         pokemons.clear();
         pokemons.addAll(fetchedPokemons);
       } catch (e) {
         print('Erro ao carregar Pokémon: $e');
       }
     }
   }
   ```

2. **Configuração do Modular:**
   - Utilize o pacote `flutter_modular` para configuração de injeção de dependência e gerenciamento de rotas.

   Exemplo de configuração (`pokemon_module.dart`):
   ```dart
   // /lib/modules/pokemon/pokemon_module.dart

   import 'package:flutter_modular/flutter_modular.dart';
   import 'package:nome_do_projeto/modules/pokemon/pokemon_page.dart';
   import 'package:nome_do_projeto/modules/pokemon/pokemon_store.dart';

   class PokemonModule extends Module {
     @override
     List<Bind> get binds => [
       Bind((_) => PokemonStore()),
     ];

     @override
     List<ModularRoute> get routes => [
       ChildRoute('/', child: (_, args) => PokemonPage()),
     ];
   }
   ```

### UI e Navegação

1. **Implementação da Interface de Usuário:**
   - Crie telas utilizando widgets Flutter para exibir os Pokémon.

   Exemplo de página (`pokemon_page.dart`):
   ```dart
   // /lib/modules/pokemon/pokemon_page.dart

   import 'package:flutter/material.dart';
   import 'package:flutter_mobx/flutter_mobx.dart';
   import 'package:flutter_modular/flutter_modular.dart';
   import 'package:nome_do_projeto/modules/pokemon/pokemon_store.dart';

   class PokemonPage extends StatefulWidget {
     @override
     _PokemonPageState createState() => _PokemonPageState();
   }

   class _PokemonPageState extends State<PokemonPage> {
     final PokemonStore store = Modular.get<PokemonStore>();

     @override
     void initState() {
       super.initState();
       store.fetchPokemons();
     }

     @override
     Widget build(BuildContext context) {
       return Scaffold(
         appBar: AppBar(
           title: Text('Gerenciador de Pokémon'),
         ),
         body: Observer(
           builder: (_) {
             if (store.pokemons.isEmpty) {
               return Center(child: CircularProgressIndicator());
             }
             return ListView.builder(
               itemCount: store.pokemons.length,
               itemBuilder: (context, index) {
                 final pokemon = store.pokemons[index];
                 return ListTile(
                   title: Text(pokemon.name),
                   // Implemente a navegação para detalhes do Pokémon
                   onTap: () {
                     // Implemente a navegação para detalhes do Pokémon
                   },
                 );
               },
             );
           },
         ),
       );
     }
   }
   ```

### Conclusão

Desenvolver um aplicativo de Gerenciador de Cards de Pokémon com Flutter envolve integrar uma API para obter dados, utilizar MobX para gerenciamento de estado reativo e Modular para organização do projeto com injeção de dependência e gerenciamento de rotas. Este projeto não apenas demonstra a implementação prática desses conceitos, mas também ressalta a importância de uma arquitetura bem estruturada para aplicativos móveis que visam escalabilidade e manutenibilidade.
