var ADMIN_NOME  = 'admin';
var ADMIN_SENHA = 'vni@2026';
function ehAdmin() { return localStorage.getItem('vni_sessao') === ADMIN_NOME; }

var estabelecimentos = [];
var tipoAtivo = 'todos';
var editandoIndex = null;

function filtrarPorTipo(tipo, botao) {
  tipoAtivo = tipo;
  document.querySelectorAll('.btn-filtro').forEach(function(b) { b.classList.remove('ativo'); });
  botao.classList.add('ativo');
  aplicarFiltros();
}

function aplicarFiltros() {
  var busca = document.getElementById('busca').value.toLowerCase();
  var filtrados = estabelecimentos.filter(function(e) {
    if (e.status === 'pendente') return false;
    var matchTipo  = tipoAtivo === 'todos' || e.tipo === tipoAtivo;
    var matchBusca = !busca ||
      e.nome.toLowerCase().includes(busca) ||
      e.categoria.toLowerCase().includes(busca) ||
      e.descricao.toLowerCase().includes(busca);
    return matchTipo && matchBusca;
  });
  renderizar(filtrados);
}

function estaAberto(neg) {
  if (neg.tipo !== 'loja') return null;
  if (!neg.abertura || !neg.fechamento || !neg.dias || neg.dias.length === 0) return null;
  var agora = new Date();
  var nomes = ['Dom','Seg','Ter','Qua','Qui','Sex','Sab'];
  if (!neg.dias.includes(nomes[agora.getDay()])) return false;
  var minAtual = agora.getHours() * 60 + agora.getMinutes();
  var partsA = neg.abertura.split(':');
  var partsF = neg.fechamento.split(':');
  var minAbre  = parseInt(partsA[0]) * 60 + parseInt(partsA[1]);
  var minFecha = parseInt(partsF[0]) * 60 + parseInt(partsF[1]);
  return minAtual >= minAbre && minAtual < minFecha;
}

function badgeHorario(neg) {
  var s = estaAberto(neg);
  if (s === null) return '';
  if (s) return '<span class="status-aberto">Aberto agora</span>';
  return '<span class="status-fechado">Fechado</span>';
}

function toggleCard(idx) {
  var det  = document.getElementById('detalhes-' + idx);
  var seta = document.getElementById('seta-' + idx);
  if (!det) return;
  var aberto = det.classList.toggle('expandido');
  if (seta) seta.textContent = aberto ? '\u25B2' : '\u25BC';
}

function destacarTexto(texto, busca) {
  if (!busca) return texto;
  var regex = new RegExp("(" + busca + ")", "gi");
  return texto.replace(regex, '<span class="highlight">$1</span>');
}

function montarCard(e, idx) {
  var busca = document.getElementById('busca').value.toLowerCase();
  var tel = e.telefone.replace(/\D/g, '');
  var msg = encodeURIComponent('Ola, vi seu negocio no Onde Tem VNI!');
  var temDetalhes = !!(e.endereco || e.cidades ||
    (e.tipo === 'loja' && e.dias && e.dias.length > 0) || !!e.maps);

  var iconeTag  = e.tipo === 'loja' ? '&#128722;' : '&#128295;';
  var labelTipo = e.tipo === 'loja' ? 'Loja' : 'Servico';

  var det = '';
  if (e.endereco) det += e.maps
  ? '<p class="card-detalhe"><a href="' + e.maps + '" target="_blank" class="link-maps">&#128205; ' + e.endereco + '</a></p>'
  : '<p class="card-detalhe">&#128205; ' + e.endereco + '</p>';

  if (e.cidades)  det += '<p class="card-detalhe">Atende: ' + e.cidades + '</p>';
  if (e.tipo === 'loja' && e.dias && e.dias.length > 0) {
    det += '<p class="card-detalhe">&#128337; ' + e.dias.join(', ') + ' &bull; ' + e.abertura + ' - ' + e.fechamento + '</p>';
  }

  var cardClass     = 'card card-' + e.tipo;
  var seta          = temDetalhes ? '<span class="seta-expand" id="seta-' + idx + '">&#9660;</span>' : '';
  var resumoOnclick = temDetalhes ? ' onclick="toggleCard(' + idx + ')" style="cursor:pointer"' : '';
  var detDiv        = temDetalhes ? '<div class="card-detalhes" id="detalhes-' + idx + '">' + det + '</div>' : '';

  return '<div class="' + cardClass + '">' +
    '<div class="card-resumo"' + resumoOnclick + '>' +
    '<div class="card-info">' +
      '<div class="card-topo">' +
        '<div>' +
          '<h2>' + destacarTexto(e.nome, busca) + '</h2>' +
          '<div style="margin-top:4px">' +
            '<span class="tag tag-' + e.tipo + '">' + iconeTag + ' ' + labelTipo + '</span>' +
            '<span class="tag tag-categoria">' + destacarTexto(e.categoria, busca) + '</span>' +
            badgeHorario(e) +
          '</div>' +
        '</div>' +
        seta +
      '</div>' +
      '<p class="card-desc">' + destacarTexto(e.descricao, busca) + '</p>' +
    '</div>' +
    '<div class="card-acoes">' +
      '<a class="btn-wpp" href="https://wa.me/55' + tel + '?text=' + msg + '" target="_blank" onclick="registrarCliqueWpp(\'' + e.id + '\');event.stopPropagation()">&#128172; WhatsApp</a>' +
      '<a class="btn-tel" href="tel:' + e.telefone + '" onclick="event.stopPropagation()">&#128222; Ligar</a>' +

    '</div>' +
    '</div>' +
    detDiv +
  '</div>';
}

function renderizar(lista) {
  var container = document.getElementById('lista-estabelecimentos');
  var contador  = document.getElementById('contador-resultados');
  contador.textContent = lista.length + ' estabelecimento(s) encontrado(s)';
  if (lista.length === 0) {
    container.innerHTML = '<p class="sem-resultado">Nenhum estabelecimento cadastrado ainda.</p>';
    return;
  }
  var html = '';
  for (var i = 0; i < lista.length; i++) {
    var idxReal = estabelecimentos.indexOf(lista[i]);
    html += montarCard(lista[i], idxReal);
  }
  container.innerHTML = html;
}

function adminEditar(idx) {
  var neg = estabelecimentos[idx];
  if (!neg) return;
  editandoIndex = idx;
  document.getElementById('titulo-negocio').textContent     = 'Editar negocio (admin)';
  document.getElementById('btn-salvar-negocio').textContent = 'Salvar alteracoes';
  document.getElementById('neg-nome').value      = neg.nome;
  document.getElementById('neg-tipo').value      = neg.tipo;
  document.getElementById('neg-tipo').disabled   = true;
  document.getElementById('neg-categoria').value = neg.categoria;
  document.getElementById('neg-descricao').value = neg.descricao;
  document.getElementById('neg-telefone').value  = neg.telefone;
  document.getElementById('neg-maps').value      = neg.maps || '';
  document.getElementById('erro-negocio').textContent = '';
  alternarCamposTipo();
  if (neg.tipo === 'loja') {
    document.getElementById('neg-endereco').value   = neg.endereco   || '';
    document.getElementById('neg-abertura').value   = neg.abertura   || '';
    document.getElementById('neg-fechamento').value = neg.fechamento || '';
    document.querySelectorAll('input[name="dia"]').forEach(function(cb) {
      cb.checked = !!(neg.dias && neg.dias.includes(cb.value));
    });
  }
  if (neg.tipo === 'servico') document.getElementById('neg-cidades').value = neg.cidades || '';
  document.getElementById('modal-negocio').classList.add('visivel');
  document.getElementById('overlay-negocio').classList.add('visivel');
}

function adminExcluir(idx) {
  if (!confirm('Excluir este cadastro?')) return;
  var neg = estabelecimentos[idx];
  estabelecimentos.splice(idx, 1);
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  salvos = salvos.filter(function(s) { return !(s.dono === neg.dono && s.tipo === neg.tipo); });
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  aplicarFiltros();
}

function alternarCamposTipo() {
  var tipo      = document.getElementById('neg-tipo').value;
  var campoEnd  = document.getElementById('campo-endereco');
  var campoCid  = document.getElementById('campo-cidades');
  var campoHor  = document.getElementById('campo-horario');
  var campoMaps = document.getElementById('campo-maps');
  var inpEnd    = document.getElementById('neg-endereco');
  var inpCid    = document.getElementById('neg-cidades');
  campoEnd.classList.add('escondido');  inpEnd.required = false;
  campoCid.classList.add('escondido');  inpCid.required = false;
  campoHor.classList.add('escondido');
  campoMaps.classList.add('escondido');
  if (tipo === 'loja') {
    campoEnd.classList.remove('escondido');  inpEnd.required = true;
    campoHor.classList.remove('escondido');
    campoMaps.classList.remove('escondido');
  } else if (tipo === 'servico') {
    campoCid.classList.remove('escondido');  inpCid.required = true;
    campoMaps.classList.remove('escondido');
  }
}

function abrirModal() {
  document.getElementById('modal-auth').classList.add('visivel');
  document.getElementById('overlay-auth').classList.add('visivel');
}
function fecharModal() {
  document.getElementById('modal-auth').classList.remove('visivel');
  document.getElementById('overlay-auth').classList.remove('visivel');
  document.getElementById('erro-login').textContent    = '';
  document.getElementById('erro-cadastro').textContent = '';
}
function trocarAba(aba) {
  var fLogin    = document.getElementById('form-login');
  var fCadastro = document.getElementById('form-cadastro');
  var aLogin    = document.getElementById('aba-login');
  var aCadastro = document.getElementById('aba-cadastro');
  if (aba === 'login') {
    fLogin.classList.remove('escondido'); fCadastro.classList.add('escondido');
    aLogin.classList.add('ativa');        aCadastro.classList.remove('ativa');
  } else {
    fLogin.classList.add('escondido');    fCadastro.classList.remove('escondido');
    aLogin.classList.remove('ativa');     aCadastro.classList.add('ativa');
  }
}

function abrirModalNegocio() {
  editandoIndex = null;
  document.getElementById('titulo-negocio').textContent     = 'Cadastrar negocio';
  document.getElementById('btn-salvar-negocio').textContent = 'Cadastrar negocio';
  document.getElementById('neg-tipo').disabled = false;
  document.getElementById('form-negocio').reset();
  document.getElementById('erro-negocio').textContent = '';
  document.getElementById('campo-endereco').classList.add('escondido');
  document.getElementById('campo-cidades').classList.add('escondido');
  document.getElementById('campo-horario').classList.add('escondido');
  document.getElementById('campo-maps').classList.add('escondido');
  document.getElementById('modal-negocio').classList.add('visivel');
  document.getElementById('overlay-negocio').classList.add('visivel');
}
function fecharModalNegocio() {
  document.getElementById('modal-negocio').classList.remove('visivel');
  document.getElementById('overlay-negocio').classList.remove('visivel');
  document.getElementById('erro-negocio').textContent = '';
  document.getElementById('form-negocio').reset();
  document.getElementById('campo-endereco').classList.add('escondido');
  document.getElementById('campo-cidades').classList.add('escondido');
  document.getElementById('campo-horario').classList.add('escondido');
  document.getElementById('campo-maps').classList.add('escondido');
  editandoIndex = null;
}

function salvarNegocio(ev) {
  ev.preventDefault();
  var nome      = document.getElementById('neg-nome').value.trim();
  var tipo      = document.getElementById('neg-tipo').value;
  var endereco  = document.getElementById('neg-endereco').value.trim();
  var cidades   = document.getElementById('neg-cidades').value.trim();
  var categoria = document.getElementById('neg-categoria').value.trim();
  var descricao = document.getElementById('neg-descricao').value.trim();
  var telefone  = document.getElementById('neg-telefone').value.trim();
  var maps      = document.getElementById('neg-maps').value.trim();
  var erro      = document.getElementById('erro-negocio');
  var dono      = localStorage.getItem('vni_sessao');
  var abertura   = tipo === 'loja' ? document.getElementById('neg-abertura').value   : '';
  var fechamento = tipo === 'loja' ? document.getElementById('neg-fechamento').value : '';
  var dias = [];
  if (tipo === 'loja') {
    document.querySelectorAll('input[name="dia"]:checked').forEach(function(cb) { dias.push(cb.value); });
  }
  if (!tipo) { erro.textContent = 'Selecione o tipo.'; return; }
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');

  if (editandoIndex !== null) {
    var negocio = {
      id: estabelecimentos[editandoIndex].id,
      nome:nome, tipo:tipo, endereco:endereco, cidades:cidades,
      categoria:categoria, descricao:descricao, telefone:telefone,
      maps:maps, abertura:abertura, fechamento:fechamento, dias:dias, dono:dono,
      status: estabelecimentos[editandoIndex].status || 'aprovado'
    };
    estabelecimentos[editandoIndex] = negocio;
    for (var i = 0; i < salvos.length; i++) {
      if (salvos[i].id === negocio.id) { salvos[i] = negocio; break; }
    }
    localStorage.setItem('vni_negocios', JSON.stringify(salvos));
    fecharModalNegocio(); aplicarFiltros(); return;
  }

  var jatem = false;
  for (var j = 0; j < salvos.length; j++) {
    if (salvos[j].dono === dono && salvos[j].tipo === tipo) { jatem = true; break; }
  }
  if (jatem) { erro.textContent = 'Voce ja cadastrou um ' + tipo + '. Edite pelo seu perfil.'; return; }

  var negocio = {
    id: gerarId(),
    nome:nome, tipo:tipo, endereco:endereco, cidades:cidades,
    categoria:categoria, descricao:descricao, telefone:telefone,
    maps:maps, abertura:abertura, fechamento:fechamento, dias:dias, dono:dono,
    status: 'pendente'
  };
  estabelecimentos.push(negocio);
  salvos.push(negocio);
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  var meus = salvos.filter(function(s) { return s.dono === dono; });
  var tl = false, ts = false;
  for (var k = 0; k < meus.length; k++) {
    if (meus[k].tipo === 'loja')    tl = true;
    if (meus[k].tipo === 'servico') ts = true;
  }
  if (tl && ts) document.getElementById('btn-add-negocio').classList.add('escondido');
  fecharModalNegocio();
  aplicarFiltros();
  alert('Cadastro enviado! Aguarde aprovacao do administrador.');
}

function abrirPerfil() {
  var sessao   = localStorage.getItem('vni_sessao');
  var salvos   = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  var meus     = salvos.filter(function(s) { return s.dono === sessao; });
  var conteudo = document.getElementById('perfil-conteudo');
  if (meus.length === 0) {
    conteudo.innerHTML = '<p class="perfil-vazio">Voce ainda nao cadastrou nenhum negocio.</p>';
  } else {
    var h = '';
    for (var i = 0; i < meus.length; i++) {
      var neg = meus[i];
      var badgeStatus = neg.status === 'pendente'
        ? '<span class="badge-pendente">&#9679; Aguardando aprovacao</span>'
        : '<span class="badge-aprovado">&#9679; Aprovado</span>';
      var diasInfo = (neg.tipo === 'loja' && neg.dias && neg.dias.length > 0)
        ? '<p>&#128337; ' + neg.dias.join(', ') + ' &bull; ' + neg.abertura + ' - ' + neg.fechamento + '</p>' : '';
      h += '<div class="perfil-item">' +
        '<h3>' + neg.nome + '</h3>' +
        '<p>' + (neg.tipo === 'loja' ? '&#128722; Loja' : '&#128295; Servico') + ' &middot; ' + neg.categoria + '</p>' +
        (neg.endereco ? '<p>&#128205; ' + neg.endereco + '</p>' : '') +
        (neg.cidades  ? '<p>Atende: ' + neg.cidades + '</p>' : '') +
        diasInfo + badgeStatus +
        '<div style="display:flex;gap:8px;margin-top:8px">' +
        '<button class="btn-editar" onclick="abrirEdicao(\'' + neg.tipo + '\')">Editar</button>' +
        '<button class="btn-excluir" onclick="excluirNegocio(\'' + neg.tipo + '\')">Excluir</button>' +
        '</div></div>';
    }
    conteudo.innerHTML = h;
  }
  document.getElementById('modal-perfil').classList.add('visivel');
  document.getElementById('overlay-perfil').classList.add('visivel');
}
function fecharPerfil() {
  document.getElementById('modal-perfil').classList.remove('visivel');
  document.getElementById('overlay-perfil').classList.remove('visivel');
}
function excluirNegocio(tipo) {
  if (!confirm('Tem certeza que deseja excluir este cadastro?')) return;
  var sessao = localStorage.getItem('vni_sessao');
  for (var i = 0; i < estabelecimentos.length; i++) {
    if (estabelecimentos[i].dono === sessao && estabelecimentos[i].tipo === tipo) {
      estabelecimentos.splice(i, 1); break;
    }
  }
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  salvos = salvos.filter(function(s) { return !(s.dono === sessao && s.tipo === tipo); });
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  document.getElementById('btn-add-negocio').classList.remove('escondido');
  aplicarFiltros(); abrirPerfil();
}
function abrirEdicao(tipo) {
  var sessao = localStorage.getItem('vni_sessao');
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  var neg = null;
  for (var i = 0; i < salvos.length; i++) {
    if (salvos[i].dono === sessao && salvos[i].tipo === tipo) { neg = salvos[i]; break; }
  }
  if (!neg) return;
  for (var j = 0; j < estabelecimentos.length; j++) {
    if (estabelecimentos[j].dono === sessao && estabelecimentos[j].tipo === tipo) {
      editandoIndex = j; break;
    }
  }
  document.getElementById('titulo-negocio').textContent     = 'Editar negocio';
  document.getElementById('btn-salvar-negocio').textContent = 'Salvar alteracoes';
  document.getElementById('neg-nome').value      = neg.nome;
  document.getElementById('neg-tipo').value      = neg.tipo;
  document.getElementById('neg-tipo').disabled   = true;
  document.getElementById('neg-categoria').value = neg.categoria;
  document.getElementById('neg-descricao').value = neg.descricao;
  document.getElementById('neg-telefone').value  = neg.telefone;
  document.getElementById('neg-maps').value      = neg.maps || '';
  document.getElementById('erro-negocio').textContent = '';
  alternarCamposTipo();
  if (tipo === 'loja') {
    document.getElementById('neg-endereco').value   = neg.endereco   || '';
    document.getElementById('neg-abertura').value   = neg.abertura   || '';
    document.getElementById('neg-fechamento').value = neg.fechamento || '';
    document.querySelectorAll('input[name="dia"]').forEach(function(cb) {
      cb.checked = !!(neg.dias && neg.dias.includes(cb.value));
    });
  }
  if (tipo === 'servico') document.getElementById('neg-cidades').value = neg.cidades || '';
  fecharPerfil();
  document.getElementById('modal-negocio').classList.add('visivel');
  document.getElementById('overlay-negocio').classList.add('visivel');
}

function obterUsuarios() { return JSON.parse(localStorage.getItem('vni_usuarios') || '[]'); }
function salvarUsuarios(u) { localStorage.setItem('vni_usuarios', JSON.stringify(u)); }

function realizarCadastro(ev) {
  ev.preventDefault();
  var nome     = document.getElementById('cad-nome').value.trim();
  var telefone = document.getElementById('cad-telefone').value.trim();
  var senha    = document.getElementById('cad-senha').value;
  var erro     = document.getElementById('erro-cadastro');
  if (nome.toLowerCase() === ADMIN_NOME) { erro.textContent = 'Nome invalido.'; return; }
  var usuarios = obterUsuarios();
  for (var i = 0; i < usuarios.length; i++) {
    if (usuarios[i].nome.toLowerCase() === nome.toLowerCase()) {
      erro.textContent = 'Ja existe uma conta com esse nome.'; return;
    }
  }
  usuarios.push({ nome:nome, telefone:telefone, senha:senha });
  salvarUsuarios(usuarios);
  logarUsuario(nome);
}
function realizarLogin(ev) {
  ev.preventDefault();
  var nome  = document.getElementById('login-nome').value.trim();
  var senha = document.getElementById('login-senha').value;
  var erro  = document.getElementById('erro-login');
  if (nome === ADMIN_NOME && senha === ADMIN_SENHA) { logarUsuario(ADMIN_NOME); return; }
  var usuarios = obterUsuarios();
  var usuario = null;
  for (var i = 0; i < usuarios.length; i++) {
    if (usuarios[i].nome.toLowerCase() === nome.toLowerCase() && usuarios[i].senha === senha) {
      usuario = usuarios[i]; break;
    }
  }
  if (!usuario) { erro.textContent = 'Nome ou senha incorretos.'; return; }
  logarUsuario(usuario.nome);
}
function logarUsuario(nome) {
  localStorage.setItem('vni_sessao', nome);
  atualizarHeaderUsuario(nome);
  fecharModal();
  if (nome === ADMIN_NOME) abrirPainelAdmin();
}
function deslogar() {
  localStorage.removeItem('vni_sessao');
  document.getElementById('user-status').innerHTML =
    '<button id="btn-auth" class="btn-login-header" onclick="abrirModal()">&#128100; Entrar</button>';
  document.getElementById('btn-add-negocio').classList.add('escondido');
  aplicarFiltros();
}
function atualizarHeaderUsuario(nome) {
  var admin  = (nome === ADMIN_NOME);
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  var tl = false, ts = false;
  for (var i = 0; i < salvos.length; i++) {
    if (salvos[i].dono === nome && salvos[i].tipo === 'loja')    tl = true;
    if (salvos[i].dono === nome && salvos[i].tipo === 'servico') ts = true;
  }
  var labelNome   = admin ? '&#9881; Admin' : '&#128100; ' + nome;
  var classeExtra = admin ? ' btn-admin-label' : '';
  var onclickBtn  = admin ? 'onclick="abrirPainelAdmin()"' : 'onclick="abrirPerfil()"';
  document.getElementById('user-status').innerHTML =
    '<div class="usuario-logado">' +
    '<button class="btn-nome-perfil' + classeExtra + '" ' + onclickBtn + '>' + labelNome + '</button>' +
    '<button class="btn-sair" onclick="deslogar()">Sair</button>' +
    '</div>';
  var btnAdd = document.getElementById('btn-add-negocio');
  if (admin || (tl && ts)) { btnAdd.classList.add('escondido'); }
  else                     { btnAdd.classList.remove('escondido'); }
  aplicarFiltros();
}

var _salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
for (var _i = 0; _i < _salvos.length; _i++) { estabelecimentos.push(_salvos[_i]); }
aplicarFiltros();
var _sessao = localStorage.getItem('vni_sessao');
if (_sessao) atualizarHeaderUsuario(_sessao);

// ========== PAINEL ADMIN ==========

function gerarId() {
  return Date.now().toString(36) + Math.random().toString(36).substr(2,5);
}
function obterCliques() { return JSON.parse(localStorage.getItem('vni_cliques') || '{}'); }
function registrarCliqueWpp(id) {
  var c = obterCliques(); c[id] = (c[id]||0)+1;
  localStorage.setItem('vni_cliques', JSON.stringify(c));
}

function abrirPainelAdmin() {
  renderizarPainelAdmin();
  document.getElementById('modal-admin').classList.add('visivel');
  document.getElementById('overlay-admin').classList.add('visivel');
}
function fecharPainelAdmin() {
  document.getElementById('modal-admin').classList.remove('visivel');
  document.getElementById('overlay-admin').classList.remove('visivel');
}
function trocarAbaAdmin(aba) {
  ['stats','pendentes','lojas','servicos','usuarios'].forEach(function(a) {
    document.getElementById('admin-aba-' + a).classList.toggle('ativa', a === aba);
    document.getElementById('admin-sec-' + a).style.display = a === aba ? 'block' : 'none';
  });
}

function renderizarPainelAdmin() {
  var salvos   = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  var usuarios = obterUsuarios();
  var cliques  = obterCliques();

  document.getElementById('admin-stat-lojas').textContent     = salvos.filter(function(n){ return n.tipo==='loja'    && n.status==='aprovado'; }).length;
  document.getElementById('admin-stat-servicos').textContent  = salvos.filter(function(n){ return n.tipo==='servico' && n.status==='aprovado'; }).length;
  document.getElementById('admin-stat-pendentes').textContent = salvos.filter(function(n){ return n.status==='pendente'; }).length;
  document.getElementById('admin-stat-usuarios').textContent  = usuarios.length;

  var pend = salvos.filter(function(n){ return n.status==='pendente'; });
  document.getElementById('admin-pendentes').innerHTML = pend.length === 0
    ? '<p style="color:#999;font-size:.85rem;text-align:center;padding:12px">Nenhum pendente.</p>'
    : pend.map(function(p){
        return '<div class="admin-card-pend">' +
          '<strong>' + p.nome + '</strong> <span class="tag tag-'+p.tipo+'">'+p.tipo+'</span>' +
          '<p>' + p.categoria + ' &middot; ' + p.descricao + '</p>' +
          '<p>&#128222; ' + p.telefone + ' &middot; Dono: ' + p.dono + '</p>' +
          '<div style="display:flex;gap:8px;margin-top:10px">' +
          '<button class="btn-aprovar"  onclick="adminAprovar(\''+p.id+'\')">&#10003; Aprovar</button>' +
          '<button class="btn-rejeitar" onclick="adminRejeitar(\''+p.id+'\')">&#10005; Rejeitar</button>' +
          '</div></div>';
      }).join('');

  var lojas = salvos.filter(function(n){ return n.tipo==='loja'; });
  document.getElementById('admin-lojas').innerHTML = lojas.length === 0
    ? '<p style="color:#999;font-size:.85rem;text-align:center;padding:12px">Nenhuma loja cadastrada.</p>'
    : lojas.map(function(n){
        var badge = n.status==='pendente' ? '<span class="badge-pendente">Pendente</span>' : '<span class="badge-aprovado">Aprovado</span>';
        var c = cliques[n.id] || 0;
        return '<div class="admin-row top">' +
          '<div><strong>'+n.nome+'</strong> '+badge+'<br>' +
          '<small>'+n.categoria+' &middot; Dono: '+n.dono+'</small><br>' +
          '<span class="admin-badge-clique" style="margin-top:4px;display:inline-block">&#128172; '+c+' clique(s) no WhatsApp</span>' +
          '</div>' +
          '<div style="display:flex;gap:6px;flex-shrink:0">' +
          '<button class="btn-painel-editar"  onclick="adminEditarPainel(\''+n.id+'\')">Editar</button>' +
          '<button class="btn-painel-excluir" onclick="adminExcluirPainel(\''+n.id+'\')">Excluir</button>' +
          '</div></div>';
      }).join('');

  var servicos = salvos.filter(function(n){ return n.tipo==='servico'; });
  document.getElementById('admin-servicos').innerHTML = servicos.length === 0
    ? '<p style="color:#999;font-size:.85rem;text-align:center;padding:12px">Nenhum servico cadastrado.</p>'
    : servicos.map(function(n){
        var badge = n.status==='pendente' ? '<span class="badge-pendente">Pendente</span>' : '<span class="badge-aprovado">Aprovado</span>';
        var c = cliques[n.id] || 0;
        return '<div class="admin-row top">' +
          '<div><strong>'+n.nome+'</strong> '+badge+'<br>' +
          '<small>'+n.categoria+' &middot; Dono: '+n.dono+'</small><br>' +
          '<span class="admin-badge-clique" style="margin-top:4px;display:inline-block">&#128172; '+c+' clique(s) no WhatsApp</span>' +
          '</div>' +
          '<div style="display:flex;gap:6px;flex-shrink:0">' +
          '<button class="btn-painel-editar"  onclick="adminEditarPainel(\''+n.id+'\')">Editar</button>' +
          '<button class="btn-painel-excluir" onclick="adminExcluirPainel(\''+n.id+'\')">Excluir</button>' +
          '</div></div>';
      }).join('');

  document.getElementById('admin-usuarios').innerHTML = usuarios.length === 0
    ? '<p style="color:#999;font-size:.85rem;text-align:center;padding:12px">Nenhum usuario.</p>'
    : usuarios.map(function(u){
        var qtd = salvos.filter(function(s){ return s.dono===u.nome; }).length;
        return '<div class="admin-row">' +
          '<div><strong>&#128100; '+u.nome+'</strong><br>' +
          '<small>&#128222; '+u.telefone+' &middot; '+qtd+' negocio(s)</small></div>' +
          '<button class="btn-painel-excluir" onclick="adminExcluirUsuario(\''+u.nome+'\')">Excluir</button>' +
          '</div>';
      }).join('');
}

function adminAprovar(id) {
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  for (var i=0; i<salvos.length; i++) { if(salvos[i].id===id){ salvos[i].status='aprovado'; break; } }
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  estabelecimentos = salvos; aplicarFiltros(); renderizarPainelAdmin();
}
function adminRejeitar(id) {
  if (!confirm('Rejeitar e excluir?')) return;
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]').filter(function(n){ return n.id!==id; });
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  estabelecimentos = salvos; aplicarFiltros(); renderizarPainelAdmin();
}
function adminExcluirPainel(id) {
  if (!confirm('Excluir este negocio?')) return;
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]').filter(function(n){ return n.id!==id; });
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  estabelecimentos = salvos; aplicarFiltros(); renderizarPainelAdmin();
}
function adminEditarPainel(id) {
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]');
  var neg = null;
  for (var i=0; i<salvos.length; i++) { if(salvos[i].id===id){ neg=salvos[i]; break; } }
  if (!neg) return;
  for (var j=0; j<estabelecimentos.length; j++) { if(estabelecimentos[j].id===id){ editandoIndex=j; break; } }
  fecharPainelAdmin();
  document.getElementById('titulo-negocio').textContent     = 'Editar negocio (admin)';
  document.getElementById('btn-salvar-negocio').textContent = 'Salvar alteracoes';
  document.getElementById('neg-nome').value      = neg.nome;
  document.getElementById('neg-tipo').value      = neg.tipo;
  document.getElementById('neg-tipo').disabled   = true;
  document.getElementById('neg-categoria').value = neg.categoria;
  document.getElementById('neg-descricao').value = neg.descricao;
  document.getElementById('neg-telefone').value  = neg.telefone;
  document.getElementById('neg-maps').value      = neg.maps || '';
  document.getElementById('erro-negocio').textContent = '';
  alternarCamposTipo();
  if (neg.tipo==='loja') {
    document.getElementById('neg-endereco').value   = neg.endereco   || '';
    document.getElementById('neg-abertura').value   = neg.abertura   || '';
    document.getElementById('neg-fechamento').value = neg.fechamento || '';
    document.querySelectorAll('input[name="dia"]').forEach(function(cb){ cb.checked=!!(neg.dias&&neg.dias.includes(cb.value)); });
  }
  if (neg.tipo==='servico') document.getElementById('neg-cidades').value = neg.cidades||'';
  document.getElementById('modal-negocio').classList.add('visivel');
  document.getElementById('overlay-negocio').classList.add('visivel');
}
function adminExcluirUsuario(nome) {
  if (!confirm('Excluir usuario "'+nome+'" e todos os seus negocios?')) return;
  salvarUsuarios(obterUsuarios().filter(function(u){ return u.nome!==nome; }));
  var salvos = JSON.parse(localStorage.getItem('vni_negocios') || '[]').filter(function(n){ return n.dono!==nome; });
  localStorage.setItem('vni_negocios', JSON.stringify(salvos));
  estabelecimentos = salvos; aplicarFiltros(); renderizarPainelAdmin();
}
