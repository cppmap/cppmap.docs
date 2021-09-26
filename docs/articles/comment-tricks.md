description: C++ のプログラムで使えるコメントアウトのトリックの紹介

# コメントアウトのトリック

プログラムで使えるコメントアウトのトリックを紹介します。  
開発中のちょっとしたデバッグや、ミスの防止に活用できます。

## 範囲の ON・OFF
スラッシュ `/` の有無に応じて、範囲コメントの有効・無効を切り替えます。

#### パターン 1

=== "無効"

	``` C++
	#include <iostream>

	int main()
	{
		/**/
		int x;

		std::cin >> x;

		std::cout << x * x;
		/**/
	}
	```

=== "有効"

	``` C++
	#include <iostream>

	int main()
	{
		/**
		int x;

		std::cin >> x;

		std::cout << x * x;
		/**/
	}
	```

#### パターン 2

=== "無効"

	``` C++
	#include <iostream>

	int main()
	{
		//*
		int x;

		std::cin >> x;

		std::cout << x * x;
		//*/
	}
	```

=== "有効"

	``` C++
	#include <iostream>

	int main()
	{
		/*
		int x;

		std::cin >> x;

		std::cout << x * x;
		//*/
	}
	```


## 値の切り替え
スラッシュ `/` の有無に応じて、左右どちらかの値を選択します。

=== "左"

	``` C++
	#include <iostream>

	int main()
	{
		constexpr int N = /**/ 100 /*/ 200 /**/;

		std::cout << N << '\n';
	}
	```

=== "右"

	``` C++
	#include <iostream>

	int main()
	{
		constexpr int N = /** 100 /*/ 200 /**/;

		std::cout << N << '\n';
	}
	```


## 範囲の切り替え
値の切り替えの範囲版です。  
スラッシュ / の有無に応じて、前半、後半どちらかの範囲を選択します。

#### パターン 1

=== "前半"

	``` C++
	#include <iostream>

	int main()
	{
		/**/
		int x;

		std::cin >> x;

		std::cout << x * x;
		/*/
		int x, y;

		std::cin >> x >> y;

		std::cout << x + y;
		/**/
	}
	```

=== "後半"

	``` C++
	#include <iostream>

	int main()
	{
		/**
		int x;

		std::cin >> x;

		std::cout << x * x;
		/*/
		int x, y;

		std::cin >> x >> y;

		std::cout << x + y;
		/**/
	}
	```

#### パターン 2

=== "前半"

	``` C++
	#include <iostream>

	int main()
	{
		//*
		int x;

		std::cin >> x;

		std::cout << x * x;
		/*/
		int x, y;

		std::cin >> x >> y;

		std::cout << x + y;
		//*/
	}
	```

=== "後半"

	``` C++
	#include <iostream>

	int main()
	{
		/*
		int x;

		std::cin >> x;

		std::cout << x * x;
		/*/
		int x, y;

		std::cin >> x >> y;

		std::cout << x + y;
		//*/
	}
	```


## 行の入れ替えの防止

リファクタリング時に、コピー＆ペーストで行の順番を入れ替えてしまうことを防ぎます。

=== "元のコード"

	``` C++
	void First() {}
	void Second() {}

	int main()
	{
		First();/*
		*/Second();
	}
	```

=== "入れ替えるとエラー"

	``` C++
	void First() {}
	void Second() {}

	int main()
	{
		*/ Second();
		First();/*	
	}
	```

