# MATLAB
FASE 3 CODIGO DE MATLAB
%% RETO 3 - ALGORITMO DE CONTROL Y CARGAS

clear;
clc;
close all;



horas = 1:24;

solar = [0 0 0 0 0 5 8 11 15 19 23 25 ...
         23 20 16 12 8 4 0 0 0 0 0 0];

eolica = [8 6 10 7 12 9 11 8 13 10 7 12 ...
          9 14 11 8 10 13 9 12 7 11 8 10];

demanda = [3 2.5 3.5 2 3 6 7 8 7 9 8 10 ...
           9 8 7 9 10 12 15 15 14 4 3 2.5];



capacidad_bateria = 100;
SOC = 50;
SOC_max = 100;
SOC_min = 0;



generacion_total = zeros(1,24);
balance = zeros(1,24);
SOC_historial = zeros(1,24);

escuela = zeros(1,24);
bombeo = zeros(1,24);
alumbrado = zeros(1,24);

estado = strings(1,24);



for h = 1:24

    generacion_total(h) = solar(h) + eolica(h);

    balance(h) = generacion_total(h) - demanda(h);

    escuela(h) = 1;
    bombeo(h) = 1;
    alumbrado(h) = 1;

    if balance(h) >= 0

        SOC = SOC + (balance(h) / capacidad_bateria) * 100;

        if SOC > SOC_max
            SOC = SOC_max;
        end

        estado(h) = "Todas las cargas activas";

    else

        SOC = SOC - (abs(balance(h)) / capacidad_bateria) * 100;

        if SOC < SOC_min
            SOC = SOC_min;
        end

        if SOC > 40

            escuela(h) = 1;
            bombeo(h) = 1;
            alumbrado(h) = 1;

            estado(h) = "Todas las cargas activas";

        elseif SOC > 20

            escuela(h) = 1;
            bombeo(h) = 1;
            alumbrado(h) = 0;

            estado(h) = "Alumbrado suspendido";

        else

            escuela(h) = 1;
            bombeo(h) = 0;
            alumbrado(h) = 0;

            estado(h) = "Bombeo y alumbrado suspendidos";

        end 
    end

    SOC_historial(h) = SOC;

end

fprintf('\n');
fprintf('REPORTE DEL ALGORITMO DE CONTROL\n');
fprintf('=============================================\n');
fprintf('Hora | Solar | Eolica | Generacion | Demanda | Balance | SOC | Estado\n');
fprintf('--------------------------------------------------------------------------\n');

for h = 1:24

    fprintf('%4d | %5.1f | %6.1f | %10.1f | %7.1f | %7.1f | %5.1f%% | %s\n', ...
        h, ...
        solar(h), ...
        eolica(h), ...
        generacion_total(h), ...
        demanda(h), ...
        balance(h), ...
        SOC_historial(h), ...
        estado(h));

end

fprintf('--------------------------------------------------------------------------\n');
fprintf('SOC inicial: %.1f %%\n',50);
fprintf('SOC final:   %.1f %%\n',SOC_historial(24));


figure(1);

plot(horas, solar, '-o', 'LineWidth', 1.5);
hold on;

plot(horas, eolica, '-s', 'LineWidth', 1.5);

plot(horas, generacion_total, '-^', 'LineWidth', 2);

plot(horas, demanda, '-d', 'LineWidth', 2);

grid on;

xlabel('Hora del día');
ylabel('Potencia (kW)');

title('Generación y demanda durante 24 horas');

legend('Solar','Eólica','Generación híbrida','Demanda', ...
       'Location','best');

xlim([1 24]);


figure(2);

plot(horas, SOC_historial, '-o', 'LineWidth', 2);

hold on;

yline(40, '--', 'SOC = 40%');
yline(20, '--', 'SOC = 20%');

grid on;

xlabel('Hora del día');
ylabel('SOC (%)');

title('Estado de carga de la batería');

legend('SOC','Límite 40%','Límite 20%', ...
       'Location','best');

xlim([1 24]);
ylim([0 100]);


figure(3);

bar(horas, balance);

grid on;

xlabel('Hora del día');
ylabel('Balance energético (kW)');

title('Balance energético de la microred');

xlim([0 25]);



figure(4);

plot(horas, escuela, '-o', 'LineWidth', 1.5);
hold on;

plot(horas, bombeo, '-s', 'LineWidth', 1.5);

plot(horas, alumbrado, '-^', 'LineWidth', 1.5);

grid on;

xlabel('Hora del día');
ylabel('Estado de la carga');

title('Estado de las cargas durante 24 horas');

legend('Escuela','Bombeo','Alumbrado', ...
       'Location','best');

xlim([1 24]);
ylim([-0.1 1.1]);

yticks([0 1]);
yticklabels({'Desconectado','Activo'});
