from typing import List, Dict, Union
from datetime import datetime
from sqlite3 import Error
from typing import Any
import sqlite3

def criar_db(cur: Any) -> None:
    with open('db.sql', 'r', encoding='utf-8') as f:
        query = f.read()
    cur.executescript(query)

con = sqlite3.connect("todo-list.db")


class TodoList:
    def __init__(self, db_path: str):
        self.conn = sqlite3.connect(db_path)
        self.cursor = self.conn.cursor()
        self.criar_tabela_sala()
        self.criar_tabela_participante()
        self.criar_tabela_inscricao()
        self.criar_tabela_evento()
        self.criar_tabela_reserva()
        self.conexao = sqlite3.connect('todo-list.db')
        self.cursor = self.conexao.cursor()
        self.salas = {}  # atributo salas criado aqui
        self.eventos = []

    

    def criar_tabela_sala(self) -> None:
        """
        Cria a tabela Sala no banco de dados todo-list.db se ela ainda não existir.
        """
        self.cursor.execute('''CREATE TABLE IF NOT EXISTS Sala (
                                id_sala INTEGER PRIMARY KEY AUTOINCREMENT,
                                nome VARCHAR(255),
                                capacidade INT,
                                localizacao VARCHAR(255)
                            )''')

    def criar_tabela_participante(self) -> None:
        """
        Cria a tabela Participante no banco de dados todo-list.db se ela ainda não existir.
        """
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS Participante (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nome TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                telefone TEXT NOT NULL
            )
        ''')
        self.conn.commit()
        print("Tabela Participante criada com sucesso!")

    def criar_tabela_inscricao(self) -> None:
        """
        Cria a tabela Inscricao no banco de dados todo-list.db se ela ainda não existir.
        """
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS inscricao (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                id_evento INTEGER NOT NULL,
                id_participante INTEGER NOT NULL,
                FOREIGN KEY (id_evento) REFERENCES evento (id),
                FOREIGN KEY (id_participante) REFERENCES participante (id)
            );
        ''')
        self.conn.commit()
        print("Tabela inscricao criada com sucesso!")

    def criar_tabela_evento(self) -> None:
        """
        Cria a tabela Evento no banco de dados todo-list.db se ela ainda não existir.
        """
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS evento (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nome TEXT NOT NULL,
                descricao TEXT NOT NULL,
                data TEXT NOT NULL,
                hora TEXT NOT NULL,
                id_sala INTEGER NOT NULL,
                FOREIGN KEY (id_sala) REFERENCES sala (id)
            );
        ''')
        self.conn.commit()
        print("Tabela evento criada com sucesso!")

    def criar_tabela_reserva(self) -> None:
        """
        Cria a tabela Reserva no banco de dados todo-list.db se ela ainda não existir.
        """
        try:
            self.cursor.execute('''
                CREATE TABLE IF NOT EXISTS reserva (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    id_sala INTEGER NOT NULL,
                    id_evento INTEGER NOT NULL,
                    data TEXT NOT NULL,
                    hora_inicio TEXT NOT NULL,
                    hora_fim TEXT NOT NULL,
                    FOREIGN KEY (id_sala) REFERENCES sala (id),
                    FOREIGN KEY (id_evento) REFERENCES evento (id)
                );
            ''')
            self.conn.commit()
            print("Tabela reserva criada com sucesso!")
        except Error as error:
            print(f"Erro ao criar tabela reserva: {error}")

    def adicionar_participante(self, nome_nov: str, email_nov: str, telefone_nov: str) -> int:
        """
    Adiciona um novo participante à tabela Participante do banco de dados.

    Args:
        nome (str): O nome do novo participante.
        email (str): O e-mail do novo participante.
        telefone (str): O número de telefone do novo participante.

    Returns:
        int: O ID do novo participante adicionado.

    """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                INSERT INTO participante (nome, email, telefone) VALUES (?, ?, ?)
            """, (nome_nov, email_nov, telefone_nov))
            self.conn.commit()
            print("Participante adicionado com sucesso!")
            return cursor.lastrowid
        except Error as error:
            print(f"Erro ao adicionar participante: {error}")
            return -1

    def obter_participantes(self):
        """
    Obtém todos os participantes do banco de dados.

    Returns:
        list: Uma lista contendo todos os participantes do banco de dados.
    """
        try:
            cursor = self.conn.cursor()
            cursor.execute("SELECT * FROM participante")
            participantes = cursor.fetchall()
            print("Participantes obtidos com sucesso!")
            return participantes
        except Error as error:
            print(f"Erro ao obter participantes: {error}")

    def criar_evento(self, nom, descricao, data, hora, id_sala):
        """
    Cria um novo evento com os dados informados e retorna o seu id.

    Parâmetros:
    nome (str): O nome do evento.
    descricao (str): A descrição do evento.
    data (str): A data do evento no formato 'YYYY-MM-DD'.
    hora (str): A hora do evento no formato 'HH:MM'.
    id_sala (int): O id da sala em que o evento será realizado.

    Retorno:
    int: O id do evento criado.
    """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                INSERT INTO evento (nome, descricao, data, hora, id_sala)
                VALUES (?, ?, ?, ?, ?)
            """, (nom, descricao, data, hora, id_sala))
            evento_id = cursor.lastrowid
            self.conn.commit()
            print(f"Evento {nom} criado com sucesso com ID {evento_id}")
            return evento_id
        except Error as error:
            print(f"Erro ao criar evento: {error}")
            return -1

    def obter_eventos(self):
        """
    Obtém todos os eventos existentes no banco de dados todo-list.db.

    Returns:
        Uma lista de dicionários contendo as informações de cada evento,
        com as chaves 'id', 'nome', 'descricao', 'data', 'hora', 'id_sala'.
        Retorna None em caso de erro.
    """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                SELECT evento.id, evento.nome, evento.descricao, evento.data, evento.hora, sala.nome as nome_sala
                FROM evento
                INNER JOIN sala ON evento.id_sala = sala.id
            """)
            eventos = cursor.fetchall()
            print("Eventos obtidos com sucesso!")
            return eventos
        except Error as error:
            print(f"Erro ao obter eventos: {error}")
            return None

    def criar_sala(self, nome_n: str, capacidade: int) -> int:
        """
    Cria uma nova sala com o nome e a capacidade especificados no banco de dados Todo-List.
    Retorna o ID da nova sala criada.

    Args:
    - nome (str): nome da sala
    - capacidade (int): capacidade máxima da sala

    Returns:
    - id_sala (int): ID da nova sala criada
    """

        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                INSERT INTO sala (nome, capacidade)
                VALUES (?, ?)
            """, (nome_n, capacidade))
            sala_id = cursor.lastrowid
            self.conn.commit()
            print(f"Sala {nome_n} criada com sucesso com ID {sala_id}")
            return sala_id
        except Error as error:
            print(f"Erro ao criar sala: {error}")
            return -1

    def obter_salas(self):
        """
        Obtém todas as salas registradas no banco de dados Todo-List.

        Returns:
        - salas (list): lista de tuplas contendo as informações das salas
        """
        try:
            cursor = self.conn.cursor()
            cursor.execute("SELECT * FROM sala;")
            salas = cursor.fetchall()
            print("Salas obtidas com sucesso!")
            return salas
        except Error as error:
            print(f"Erro ao obter salas: {error}")
            return None

    def criar_inscricao(self, id_participant: int, id_event: int) -> int:
        """
    Cria uma inscrição no evento para o participante.

    Args:
        id_participante: O ID do participante.
        id_evento: O ID do evento.

    Returns:
        O ID da inscrição criada ou -1 se houver algum erro.

    """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                INSERT INTO inscricao (id_participante, id_evento) VALUES (?, ?)
            """, (id_participant, id_event))
            self.conn.commit()
            print("Inscrição criada com sucesso!")
            return cursor.lastrowid
        except Error as error:
            print(f"Erro ao criar inscrição: {error}")
            return -1

    def obter_inscricoes(self):
        """
        Retorna todas as inscrições no banco de dados.

        Returns:
            list: Uma lista de tuplas, onde cada tupla contém o ID do participante e o
              ID do evento da inscrição correspondente.
            Retorna None se ocorrer um erro ao obter as inscrições.
        """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                SELECT * FROM inscricao;
            """)
            inscricoes = cursor.fetchall()
            print("Inscrições obtidas com sucesso!")
            return inscricoes
        except Error as error:
            print(f"Erro ao obter inscrições: {error}")

    def criar_reserva(self, id_sala, data, hora):
        """
        Cria uma reserva para uma determinada sala em uma data e hora especificadas.

        Args:
            id_sala (int): ID da sala a ser reservada.
            data (str): Data da reserva no formato 'YYYY-MM-DD'.
            hora (str): Hora da reserva no formato 'HH:MM'.

        Returns:
            None
        """
        try:
            cursor = self.conn.cursor()
            cursor.execute("""
                INSERT INTO reserva (id_sala, data, hora)
                VALUES (?, ?, ?);
            """, (id_sala, data, hora))
            self.conn.commit()
            print("Reserva criada com sucesso!")
        except sqlite3.Error as error:
            print(f"Erro ao criar reserva: {error}")

    def obter_reservas(self):
        """
        Obtém todas as reservas existentes no banco de dados.

        Returns:
        List[Tuple[int, int, str, str]]: Lista de tuplas contendo as informações das reservas.
            Cada tupla contém o ID da reserva, o ID da sala,
              a data da reserva e a hora da reserva, respectivamente.
        """
        try:
            cursor = self.conn.cursor()
            cursor.execute("SELECT * FROM reserva;")
            reservas = cursor.fetchall()
            print("Reservas obtidas com sucesso:")
            for reserva in reservas:
                print(
                    f"ID: {reserva[0]}, Sala: {reserva[1]}, Data: {reserva[2]}, Hora: {reserva[3]}")
            return reservas
        except sqlite3.Error as error:
            print(f"Erro ao obter reservas: {error}")
            return []

    def obter_salas_disponiveis_para_data_hora(self, data: datetime, hora: datetime) -> list:
        """
        Obtém todas as salas disponíveis para uma determinada data e hora.
        """
        with sqlite3.connect('banco.db') as conn:
            cur = conn.cursor()
            query = '''SELECT id_sala, nome, capacidade, localizacao FROM Sala 
                    WHERE id_sala NOT IN 
                        (SELECT id_sala FROM Evento 
                            WHERE DATE(data) = ? AND TIME(data) = ?)'''
            cur.execute(query, (data.date(), hora.time()))
            return cur.fetchall()

    def obter_participantes_para_reserva(self, id_reserva: int) -> List[Dict[str, Union[int, str]]]:
        """
        Obtém todos os participantes de uma determinada reserva.
        """
        with sqlite3.connect('banco.db') as conn:
            cur = conn.cursor()
            cur.execute("""
                SELECT p.id_participante, p.nome, p.email
                FROM Participante p
                JOIN Reserva_Participante rp ON p.id_participante = rp.id_participante
                WHERE rp.id_reserva = ?
            """, (id_reserva,))
            rows = cur.fetchall()
            participantes = []
            for row in rows:
                participantes.append({
                    'id_participante': row['id_participante'],
                    'nome': row['nome'],
                    'email': row['email']
                })
            return participantes


def criar_tabela_reserva_participante(self) -> None:
    """Cria a tabela ReservaParticipante no banco de dados."""
    try:
        cursor = self.conn.cursor()
        cursor.execute("""
                CREATE TABLE ReservaParticipante (
                    id_reserva INTEGER NOT NULL,
                    id_participante INTEGER NOT NULL,
                    FOREIGN KEY (id_reserva) REFERENCES Reserva (id),
                    FOREIGN KEY (id_participante) REFERENCES Participante (id),
                    PRIMARY KEY (id_reserva, id_participante)
                )
            """)
        self.conn.commit()
        print("Tabela ReservaParticipante criada com sucesso!")
    except Error as error:
        print(f"Erro ao criar tabela ReservaParticipante: {error}")


def obter_participantes_para_reserva(self, id_reserva: int) -> List[Dict[str, Union[int, str]]]:
    """
    Obtém todos os participantes de uma determinada reserva.
    """
    try:
        cursor = self.conn.cursor()
        cursor.execute("""
                SELECT p.id_participante, p.nome
                FROM Participante p
                JOIN ReservaParticipante rp ON p.id_participante = rp.id_participante
                WHERE rp.id_reserva = ?
            """, (id_reserva,))
        rows = cursor.fetchall()
        participantes = [{'id_participante': row[0], 'nome': row[1]}
                         for row in rows]
        return participantes
    except Error as error:
        print(f"Erro ao obter participantes para reserva: {error}")


def adicionar_evento(self, nom_n, sala, data, hora_inicio, hora_fim):
    """
    Adiciona um novo evento e reserva uma sala, 
    verificando se a sala está disponível no horário solicitado.

    Args:
        nome (str): Nome do evento.
        sala (str): Nome da sala a ser reservada.
        data (str): Data do evento no formato 'yyyy-mm-dd'.
        hora_inicio (str): Hora de início do evento no formato 'hh:mm'.
        hora_fim (str): Hora de término do evento no formato 'hh:mm'.

    Returns:
        str: Uma mensagem de sucesso ou erro, dependendo do resultado da operação.

    """
    # verificar se a sala já existe
    if sala not in self.salas:
        # cria uma lista vazia para armazenar as reservas
        self.salas[sala] = []

    # criar uma nova reserva para a sala
    reserva = {'nome': nom_n, 'data': data,
               'hora_inicio': hora_inicio, 'hora_fim': hora_fim}

    # verificar se a sala está disponível no horário solicitado
    for reserva_existente in self.salas[sala]:
        if (reserva_existente['data'] == data and
                reserva_existente['hora_inicio'] < hora_fim and
                reserva_existente['hora_fim'] > hora_inicio):
            # sala já está reservada neste horário
            return "Sala já está reservada neste horário."

    # adicionar a nova reserva à lista de reservas da sala
    self.salas[sala].append(reserva)

    # adicionar o novo evento à lista de eventos
    evento = {'nome': nom_n, 'sala': sala, 'data': data,
              'hora_inicio': hora_inicio, 'hora_fim': hora_fim}
    self.eventos.append(evento)

    # retorno para indicar que a reserva e o evento foram criados com sucesso
    return "Reserva e evento criados com sucesso."


def adicionar_participante(self, nomee: str, emaiil: str, telefone_t: str) -> int:
    """Adiciona um novo participante na tabela Participante e retorna o ID gerado"""
    self.cursor.execute(
        "INSERT INTO Participante (nome, email, telefone) VALUES (?, ?, ?)",
        (nomee, emaiil, telefone_t))
    self.conexao.commit()
    return self.cursor.lastrowid


def obter_salas_disponiveis_para_data_hora(self, data: datetime, hora: datetime) -> List[Dict[str, Union[int, str]]]:
    """Obtém todas as salas disponíveis para uma determinada data e hora"""
    query = '''SELECT id_sala, nome, capacidade, localizacao FROM Sala 
                WHERE id_sala NOT IN 
                    (SELECT id_sala FROM Evento 
                    WHERE DATE(data) = ? AND TIME(data) = ?)'''
    self.cursor.execute(query, (data.date(), hora.time()))
    salas = []
    for row in self.cursor.fetchall():
        salas.append({
            'id_sala': row[0],
            'nome': row[1],
            'capacidade': row[2],
            'localizacao': row[3]
        })
    return salas
