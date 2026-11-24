# CONCLAVE

## Допълнителна SQL заявка
В Supabase → SQL Editor → New query постави и изпълни целия код:

```sql
create or replace function public.conclave_is_host()
returns boolean language sql security definer
set search_path = private, public
as $$
 select exists (
  select 1 from private.conclave_settings
  where host_code = (current_setting('request.headers', true)::jsonb ->> 'x-conclave-code')
 );
$$;
revoke all on function public.conclave_is_host() from public;
grant execute on function public.conclave_is_host() to anon, authenticated;

create table if not exists public.conclave_game_state (
 id integer primary key check (id=1),
 phase text not null default 'day' check (phase in ('day','night')),
 night_number integer not null default 0
);
insert into public.conclave_game_state(id,phase,night_number)
values(1,'day',0) on conflict(id) do nothing;
alter table public.conclave_game_state enable row level security;

create policy "Room members read game state"
on public.conclave_game_state for select to anon, authenticated
using (public.conclave_has_access());

create policy "Host updates game state"
on public.conclave_game_state for update to anon, authenticated
using (public.conclave_is_host())
with check (public.conclave_is_host());

create policy "Room members update votes"
on public.conclave_votes for update to anon, authenticated
using (public.conclave_has_access())
with check (public.conclave_has_access());
```

## Публикуване
Качи `index.html` в GitHub repository и свържи repository-то с Vercel като статичен сайт. Не добавяй Secret key или паролата на базата данни към сайта.

Тази първа версия използва периодично обновяване на чата, а не WebSocket. Тествай я с няколко устройства преди партито.
