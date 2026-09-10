# Guia — Salvar histórico (Firestore agora) → migrar para Supabase (depois)

Painel de Cotação: https://piratruck.github.io/cotacao-piratruck/
Projeto Firebase atual: **painel-veiculos-4a6f5** · coleção **cotacoes**

---

## FASE 1 — Salvar histórico AGORA (Firestore) · ~3 min · só você

O painel já grava na coleção `cotacoes`; só falta liberar a regra no console.

1. Acesse https://console.firebase.google.com e abra o projeto **painel-veiculos-4a6f5**.
2. Menu **Criação (Build) → Firestore Database → aba Regras (Rules)**.
3. Dentro do bloco `match /databases/{database}/documents {`, **antes** da linha `match /{document=**}`, adicione:

   ```
   match /cotacoes/{id} {
     allow read, write: if request.auth != null;
   }
   ```

   ⚠️ **Só adicione essas 3 linhas.** Não apague o resto — as outras coleções (orcamentos, checklist_veiculos, financas_pessoais, agenda) precisam continuar lá.

4. Clique em **Publicar (Publish)**.
5. (Confirmar 1x) **Authentication → Settings → Domínios autorizados**: `piratruck.github.io` deve estar na lista (já está, pois os outros painéis rodam nele).
6. **Teste:** abra o painel, faça login, salve uma cotação e veja em **Histórico**. Ela também aparece em **Firestore → Data → coleção `cotacoes`**.

Pronto — histórico salvando e compartilhado entre os consultores.

---

## FASE 2 — Migrar para Supabase (depois) · projeto de dev

O curso usa Supabase. Firebase e Supabase fazem a mesma coisa aqui (login + banco), então **isso é padronização, não correção** — só vale a pena se você quiser seguir o padrão do curso. Funciona 100% no Firebase hoje.

### O que VOCÊ faz (console — não tenho acesso)
1. Criar conta/projeto em https://supabase.com (free), região **São Paulo (sa-east-1)**.
2. Copiar **Project URL** e **anon public key** (Settings → API) e me mandar.
3. **Authentication → Providers → Email**: habilitar. Criar os usuários (mesmos e-mails dos consultores) OU habilitar cadastro.

### O que EU faço (código)
4. Criar a tabela no Supabase (SQL):
   ```sql
   create table cotacoes (
     id text primary key,
     usuario text,
     dados jsonb,
     criado_em timestamptz default now()
   );
   alter table cotacoes enable row level security;
   create policy "logados_leem_gravam" on cotacoes
     for all using (auth.role() = 'authenticated')
     with check (auth.role() = 'authenticated');
   ```
5. Trocar no `index.html`: SDK do Firebase → **supabase-js**.
   - Login: `signInWithEmailAndPassword` → `supabase.auth.signInWithPassword`.
   - Salvar: `db.collection('cotacoes').doc(id).set(rec)` → `supabase.from('cotacoes').upsert(rec)`.
   - Histórico ao vivo: `onSnapshot(...)` → `supabase.from('cotacoes').select()` + Realtime.
   - Excluir: `.delete()` → `supabase.from('cotacoes').delete().eq('id', id)`.
6. Migrar o histórico existente (exportar `cotacoes` do Firestore → importar no Supabase), se já houver dados.
7. Publicar e testar (mesmo GitHub Pages).

### Ordem recomendada
Primeiro FASE 1 (histórico funcionando hoje). Só parta pra FASE 2 quando decidir padronizar no Supabase — aí me manda URL + anon key que eu faço a troca do código e a migração dos dados.
