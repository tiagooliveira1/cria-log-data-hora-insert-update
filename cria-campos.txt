
DO $$
declare
  r record;
  vSQLTrigger character varying (5000);
  vSQLFunction character varying (5000);
  vPreparedSQL character varying (5000);
BEGIN
  --drop function fcn_set_date_insert_or_update();
  -- cria function para setar os novos valores para data de insert e update
  
  vSQLFunction = '
  create function fcn_set_date_insert_or_update()
  RETURNS trigger as
  $BODY$ 
	BEGIN
	  IF TG_OP = ''INSERT'' THEN
	    -- insere a data atual no campo
	    NEW.date_insert = current_timestamp;
	  ELSE
	    -- nao deixa alterar a data de insercao 
	    NEW.date_insert = OLD.date_insert;
		-- informa a data e hora atual para o campo last_update
	    NEW.last_update = current_timestamp;
	  END IF;

	  RETURN NEW;
	END; 
	
	$BODY$
	LANGUAGE plpgsql VOLATILE
  ';
  EXECUTE vSQLFunction;
  -- retorna todas as tabelas do schema 'public'
  FOR r in select table_name from information_schema.tables where table_schema = 'public' and table_type = 'BASE TABLE'
	LOOP
	  -- insere o campo na tabela para controle de insert e update
	  EXECUTE format('ALTER TABLE %s ADD COLUMN date_insert timestamp, ADD COLUMN last_update timestamp',r.table_name);
	  
	  /* -- descomente a instrucao abaixo caso queira excluir os campos criados na tabela 
	  EXECUTE format('ALTER TABLE %s DROP COLUMN date_insert',r.table_name);		
	  EXECUTE format('ALTER TABLE %s DROP COLUMN last_update',r.table_name);
	  */
	  
	  -- exclui a trigger criada para esta tabela
	  EXECUTE format('drop trigger if exists %s ON %s','trg_set_date_insert_'|| r.table_name, r.table_name || ';', r.table_name);
	  
	  -- cria corpo da trigger
	  vSQLTrigger = 'create trigger %s 
		before insert OR update on %s
		FOR EACH ROW
		EXECUTE PROCEDURE fcn_set_date_insert_or_update();
		
	  ';
	  -- executa a criacao da trigger
	  EXECUTE format(vSQLTrigger,'trg_set_date_insert_'|| r.table_name, r.table_name); 
	  
	  
	END LOOP;
END; $$;

