Instrução compatível com PostgreSQL 9.x para inserir campos de data (timestamp) em todas as tabelas da base de dados conectada,
inserindo dois campos em cada tabela (date_insert e last_update). Este script criará também uma function, responsável por 
retornar os novos valores para os campos citados, e uma trigger, que chamará a respectiva function.
