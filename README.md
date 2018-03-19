# CM
#include "stdafx.h"
#include <iostream>
#include "math.h"
#include <fstream>

using namespace std;

const int N = 10;
const double eps_0 = pow(10, -6);

//функция интерполяции
double func(double x)
{
	double y;
	y = pow(x, 3) / 1000 - 2 * x + 6;
	return y;
}

// генератор случайных чисел
double fRand(double fMin, double fMax)
{
	double f = (double)rand() / RAND_MAX;
	return fMin + f * (fMax - fMin);
}

//кубический сплайн
double spline(double* X, double* Y, double* h, double* G, double var)
{
	int j = -1;
	double spline_val = 0;
	for (int i = 0; i < N - 1; i++)
	{
		if ((var > X[i]) && (var <= X[i + 1])) j = i;
	}
	spline_val = Y[j] * (X[j + 1] - var) / h[j + 1];
	spline_val += Y[j + 1] * (var - X[j]) / h[j + 1];
	spline_val += G[j] * (pow(X[j + 1] - var, 3) - h[j + 1] * h[j + 1] * (X[j + 1] - var)) / (6 * h[j + 1]);
	spline_val += G[j + 1] * (pow(var - X[j], 3) - h[j + 1] * h[j + 1] * (var - X[j])) / (6 * h[j + 1]);
	return spline_val;
}

int main()
{
	double A = -20, B = 15;
	double X[N], Y[N], G[N], h[N - 1];

	// сетка
	double step_0 = (B - A) / (N - 1);

	for (int i = 0; i < N; i++)
	{
		X[i] = A + i * step_0;
		Y[i] = func(X[i]);
		cout << X[i] << " " << Y[i] << endl;
	} //

	for (int i = 1; i < N; i++)
	{
		h[i] = X[i] - X[i - 1];
	} //

	  //метод прогонки для (N-1) x (N-1) матрицы для G [i]

	int n = N - 1;
	double alpha[N - 1], beta[N - 1];
	alpha[1] = 0;
	beta[1] = 0;

	for (int i = 1; i < N - 1; i++) //назад
	{
		double a, b, c, d;
		a = h[i] / 6;
		b = (h[i] + h[i + 1]) / 3;
		c = h[i + 1] / 6;
		d = (Y[i + 1] - Y[i]) / h[i + 1] - (Y[i] - Y[i - 1]) / h[i];

		alpha[i + 1] = -c / (a*alpha[i] + b);
		beta[i + 1] = (d - a * beta[i]) / (a*alpha[i] + b);
		cout << a << " " << b << " " << c << " " << d << " " << endl;
	}

	G[0] = A;
	G[N - 1] = B;
	G[N] = beta[N - 1];
	for (int i = N - 1; i-- > 0;) //вперед
	{
		G[i] = alpha[i + 1] * G[i + 1] + beta[i + 1];
	}
	G[0] = A;
	//допустим работает 

	cout << endl;

	//значение сплайна

	double var = 9.4444; //или любое другое значение от A до B
	double spline_val = 0;

	cout << "for x = " << var << " S = " << spline(X, Y, h, G, var) << endl;

	//дифференциалы
	double df, d2f, x_0;
	x_0 = (A + B) / 2;
	df = (spline(X, Y, h, G, x_0 + eps_0) - spline(X, Y, h, G, x_0)) / eps_0;
	d2f = (spline(X, Y, h, G, x_0) - 2 * spline(X, Y, h, G, x_0 + eps_0) + spline(X, Y, h, G, x_0 + 2 * eps_0)) / pow(eps_0, 2);

	//in .txt file
	double r, y_spl;
	double eps = pow(10, -1);
	FILE *f1 = fopen("Spline_Graph.txt", "w");
	for (r = X[0]; r <= X[N - 1]; r = r + eps)
	{
		y_spl = spline(X, Y, h, G, r);
		fprintf(f1, "%.7Lf %.7Lf %.7Lf \n", r, func(r), y_spl);
	}
	fclose(f1);

	FILE *f2 = fopen("spline_values.txt", "w"); //открываем файл на запись
	for (int i = 0; i < N; i++)
	{
		fprintf(f2, "%.7Lf %.7Lf \n", X[i], Y[i]);
	}
	fclose(f2);

	system("pause");
	return 0;
}
