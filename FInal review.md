# CS246 Final Review

## 1.Designing OO Programs

### (a) Visitor Pattern
- Double dispatch
- A - A1, A2, ...
- B - B1, B2, ...
- specialize code for both hierarchy	

code

	class A1 : public A {
	public:
		// do NOT put this method in base class (abstract)
		// otherwise you will lose the type of A1
		virtual void accept (B &b) override {
			b.visit(*this);	
		}
	};

	class B1 : public B {
	public:
		// one overloading for each subclass
		virtual void visit (A1 &a) override  { 
		              // subclass of A, get information of A1
 		.........
		}
	};

## 6. Template Functions
`foldl` (iterative)

	template<typyname Fn, typename T, typename X-Iter> 
	T foldl (Fn f, T base, X-Iter begin, X-Iter end) {
		while (begin != end) {
			base = f (*begin, base);
			++begin;
		}
		return base;
	}

`foldr` (recursive)

	template<typename Fn, typename T, typename X-Iter>
	T foldr (Fn f, T base, X-Iter begin, X-Iter end) {
		// normally, iterator does not come with operatoe --
		if (!begin != end ) { // begin == end, == may not be implemented
			return begin;
		} else {
			XIter next = begin;
			++next;
			return  f(*begin,foldr(f, base, next, end));
		}
	}

## 2. Forward Declaration
### Forward Declaration:
- C
- D (because it is a reference)
- E, F, G (because it it just declaration of function)
- H, I, J
### Include
- A
- B
- **`<iostream>`**

## 3. Exceptions and Exception Safety

#### 1. Basic Guarantee

Assume no-throw for Node();

	Node giveMeANode(int a){
	Node n; // Node()
	try{	
		n = Node(a); // Node (int x), then copy assignment
	} catch (...){
	}	
	return n; // copy assignment
	}
#### 2. Strong Guarantee

Assume ctor of A & B have strong guarantee

	void C::f() {
		auto temp = make_unique<CImpl>(*pImpl);
		// if copy assignment throws an exception, no changes has make to pImpl
		temp->a.method1(); // No-throw
		temp->b.method2(); // No-throw
		std::swap(pImpl, temp); // No-throw
	}

#### 3. No Throw or No Guarantee 
	// SEG fault

#### 3.1 No Guarantee
	void g(){
		int *x = new Int;
		*x = 3; 
		delete x;
	}

#### 4. Basic Guarantee

	std::swap(d, other.d); // it may throws!


## 5. Iterator

without pImpl

	template <typename T>
	class BinTree {
		T data;	
		BinTree *left, *right;
		BinTree *parent;
		public {		
		~~~~~~~~~
		}	
	};


.h

	template <typename T>
	class BinTree{
		struct BinNode;
		unique_ptr<BinNode> node; // if nullptr, node DNE
	public:
		~~~~
	};

.cc 

	struct BinTree::BinNode{
		T data;
		BinNode *left, *right, *parent;
	};

Iterator

	BinTree<T>:: Iterator {
		BinTree *curr;
		BinTree<T>::Iterator(BinTree *b): curr{b}{} //ctor
	public:
		bool operator != (const Iterator &other) {
			return curr != other.curr;
		}

		T &operator*() {
			return curr->node->data;
		}

		Iterator &operator++() {
			if(curr->node->right) {
				curr = curr->node->right;
				while (curr->node->left) {
					curr = curr->node->left;
				}
			} else {
				while (curr->node->parent && curr == curr->node->parent->node->right) {
					curr = curr->node->parent;
				}
				curr = curr->node->parent;
			}
			return *this;
		}
		friend class BinTree<T>;
	};

	Iterator begin() {
		BinTree* curr {*this};
		while (curr->node->left) curr = curr->node->left;
		return Iterator{curr};
	}
	
	Iterator end() {
		return nullptr{};
	}

	