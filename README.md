from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow


app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:root@localhost/Quadro_de_Notas'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False


db = SQLAlchemy(app)
ma = Marshmallow(app)

# ====================
# MODELOS (CLASSES ORM)
# ====================


class Estudante(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nome = db.Column(db.String(100), nullable=False)
    nota1 = db.Column(db.Float, nullable=False)
    nota2 = db.Column(db.Float, nullable=False)
    turma = db.Column(db.String(50), nullable=False)


class UsuarioAdmin(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True, nullable=False)
    password = db.Column(db.String(100), nullable=False)

# ====================
# SCHEMAS PARA SERIALIZAÇÃO
# ====================


class EstudanteSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Estudante
        load_instance = True


class UsuarioAdminSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = UsuarioAdmin
        load_instance = True


estudante_schema = EstudanteSchema()
estudantes_schema = EstudanteSchema(many=True)

usuario_admin_schema = UsuarioAdminSchema()
usuarios_admin_schema = UsuarioAdminSchema(many=True)

# ====================
# ENDPOINTS RESTFUL - CRUD
# ====================


@app.route('/estudantes', methods=['POST'])
def inserir_estudante():
    dados = request.json
    novo_estudante = Estudante(**dados)
    db.session.add(novo_estudante)
    db.session.commit()
    return estudante_schema.jsonify(novo_estudante)


@app.route('/estudantes', methods=['GET'])
def listar_estudantes():
    estudantes = Estudante.query.all()
    return estudantes_schema.jsonify(estudantes)


@app.route('/estudantes/<int:id>', methods=['GET'])
def procurar_estudante(id):
    estudante = Estudante.query.get_or_404(id)
    return estudante_schema.jsonify(estudante)


@app.route('/estudantes/<int:id>', methods=['PUT'])
def alterar_estudante(id):
    estudante = Estudante.query.get_or_404(id)
    dados = request.json
    for chave, valor in dados.items():
        setattr(estudante, chave, valor)
    db.session.commit()
    return estudante_schema.jsonify(estudante)


@app.route('/estudantes/<int:id>', methods=['DELETE'])
def apagar_estudante(id):
    estudante = Estudante.query.get_or_404(id)
    db.session.delete(estudante)
    db.session.commit()
    return jsonify({"message": "Estudante removido com sucesso."})


@app.route('/usuarios', methods=['POST'])
def inserir_usuario():
    dados = request.json
    novo_usuario = UsuarioAdmin(**dados)
    db.session.add(novo_usuario)
    db.session.commit()
    return usuario_admin_schema.jsonify(novo_usuario)


@app.route('/usuarios', methods=['GET'])
def listar_usuarios():
    usuarios = UsuarioAdmin.query.all()
    return usuarios_admin_schema.jsonify(usuarios)


@app.route('/usuarios/<int:id>', methods=['GET'])
def procurar_usuario(id):
    usuario = UsuarioAdmin.query.get_or_404(id)
    return usuario_admin_schema.jsonify(usuario)


@app.route('/usuarios/<int:id>', methods=['PUT'])
def alterar_usuario(id):
    usuario = UsuarioAdmin.query.get_or_404(id)
    dados = request.json
    for chave, valor in dados.items():
        setattr(usuario, chave, valor)
    db.session.commit()
    return usuario_admin_schema.jsonify(usuario)


@app.route('/usuarios/<int:id>', methods=['DELETE'])
def apagar_usuario(id):
    usuario = UsuarioAdmin.query.get_or_404(id)
    db.session.delete(usuario)
    db.session.commit()
    return jsonify({"message": "Usuário administrador removido com sucesso."})

# ====================
# CRIAÇÃO DE TABELAS
# ====================

@app.before_first_request
def create_tables():
    db.create_all()

# ====================
# EXECUÇÃO DO SERVIDOR
# ====================

if __name__ == '__main__':
    app.run(debug=True)
