# **📝 React Forms & Type Safety - Complete Guide**

## **🎯 Understanding Forms in React**

### **Concept (70%)**
Forms are **critical but complex** parts of web applications. They involve **state management, validation, submission, and user feedback**. React forms have evolved from controlled components to powerful form libraries.

**Form Challenges:**
1. **State management** - Tracking multiple input values
2. **Validation** - Client-side and server-side validation
3. **User experience** - Real-time feedback, error handling
4. **Performance** - Preventing unnecessary re-renders
5. **Accessibility** - Proper labels, error announcements

**The Form Evolution:**
```
Controlled Components → Formik → React Hook Form
(Complex)             → (Easier) → (Performant)
```

---

## **1️⃣ TypeScript - Type Safety Foundation**

### **Concept (70%)**
TypeScript is a **typed superset of JavaScript** that adds static types. For forms, it provides **compile-time validation** of your form data structures.

**Why TypeScript for Forms:**
- 🎯 **Catch errors early** - During development, not runtime
- 📝 **Self-documenting** - Types describe data structures
- 🔧 **Better IDE support** - Autocomplete, refactoring
- 🛡️ **Runtime safety** - Type guards prevent invalid states

**TypeScript Form Patterns:**
1. **Interface/Type definitions** for form data
2. **Generic components** for reusable form elements
3. **Type guards** for runtime validation
4. **Utility types** for partial updates, validation schemas

### **Code (30%)**
```typescript
// 1. Basic Type Definitions
interface UserFormData {
  id?: number;
  name: string;
  email: string;
  age: number;
  preferences: {
    newsletter: boolean;
    theme: 'light' | 'dark';
  };
}

// Using Type Alias
type UserFormData = {
  id?: number;
  name: string;
  email: string;
  age: number;
};

// 2. Utility Types for Forms
type FormErrors<T> = {
  [K in keyof T]?: string;
};

type FormTouched<T> = {
  [K in keyof T]?: boolean;
};

type FormStatus = 'idle' | 'submitting' | 'success' | 'error';

// 3. Generic Form Component
interface FormFieldProps<T> {
  name: keyof T;
  label: string;
  type?: string;
  required?: boolean;
}

function FormField<T>({ name, label, type = 'text' }: FormFieldProps<T>) {
  return (
    <div>
      <label htmlFor={String(name)}>{label}</label>
      <input 
        id={String(name)} 
        name={String(name)} 
        type={type} 
      />
    </div>
  );
}

// 4. Type Guards for Validation
function isUserFormData(data: unknown): data is UserFormData {
  return (
    typeof data === 'object' &&
    data !== null &&
    'name' in data &&
    'email' in data &&
    'age' in data
  );
}

// 5. Form State with TypeScript
interface FormState<T> {
  values: T;
  errors: FormErrors<T>;
  touched: FormTouched<T>;
  status: FormStatus;
}

function useForm<T extends Record<string, any>>(initialValues: T) {
  const [state, setState] = useState<FormState<T>>({
    values: initialValues,
    errors: {},
    touched: {},
    status: 'idle',
  });

  const setFieldValue = useCallback(<K extends keyof T>(
    field: K,
    value: T[K]
  ) => {
    setState(prev => ({
      ...prev,
      values: { ...prev.values, [field]: value },
    }));
  }, []);

  return { state, setFieldValue };
}

// 6. API Response Types
interface ApiResponse<T> {
  data?: T;
  error?: string;
  success: boolean;
}

async function submitForm(data: UserFormData): Promise<ApiResponse<User>> {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify(data),
  });
  
  return response.json();
}

// 7. Using with React components
interface SignupFormProps {
  onSubmit: (data: UserFormData) => void;
}

function SignupForm({ onSubmit }: SignupFormProps) {
  const [formData, setFormData] = useState<UserFormData>({
    name: '',
    email: '',
    age: 0,
    preferences: {
      newsletter: false,
      theme: 'light',
    },
  });

  // Type-safe event handlers
  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>
  ) => {
    const { name, value, type } = e.target;
    
    setFormData(prev => {
      if (type === 'checkbox') {
        const checked = (e.target as HTMLInputElement).checked;
        return { ...prev, [name]: checked };
      }
      
      if (name in prev) {
        const key = name as keyof UserFormData;
        return { ...prev, [key]: value };
      }
      
      return prev;
    });
  };

  return <form>{/* ... */}</form>;
}
```

---

## **2️⃣ Zod - Runtime Validation & Type Inference**

### **Concept (70%)**
Zod is a **TypeScript-first schema declaration and validation library**. It allows you to **define schemas once** and get both **TypeScript types and runtime validation**.

**Zod Core Concepts:**
- 🎯 **Single source of truth** - Schema defines both types and validation
- 🔄 **Type inference** - Automatically generates TypeScript types
- 🛡️ **Runtime validation** - Validate data at runtime
- 🎭 **Composable** - Build complex schemas from simple ones
- ⚡ **Zero dependencies** - Small bundle size

**Zod vs Manual Validation:**
- **Manual**: Write types, then write separate validation logic
- **Zod**: Write schema once, get both types and validation

### **Code (30%)**
```typescript
import { z } from 'zod';

// 1. Basic Schema Definition
const userSchema = z.object({
  id: z.number().optional(),
  name: z.string()
    .min(2, 'Name must be at least 2 characters')
    .max(50, 'Name must be less than 50 characters'),
  email: z.string()
    .email('Invalid email address'),
  age: z.number()
    .min(18, 'Must be at least 18 years old')
    .max(120, 'Are you really that old?'),
  birthdate: z.coerce.date() // Auto-convert string to Date
    .min(new Date('1900-01-01'), 'Too old')
    .max(new Date(), 'Cannot be born in the future'),
  website: z.string()
    .url('Invalid URL')
    .optional()
    .or(z.literal('')), // Allow empty string
  preferences: z.object({
    newsletter: z.boolean().default(false),
    theme: z.enum(['light', 'dark', 'auto']).default('auto'),
  }),
  tags: z.array(z.string()).min(1, 'Select at least one tag'),
});

// 2. Type Inference (automatically!)
type User = z.infer<typeof userSchema>;
// Equivalent to:
// type User = {
//   id?: number;
//   name: string;
//   email: string;
//   age: number;
//   birthdate: Date;
//   website?: string;
//   preferences: { newsletter: boolean; theme: 'light' | 'dark' | 'auto' };
//   tags: string[];
// }

// 3. Validation
try {
  const validUser = userSchema.parse({
    name: 'John',
    email: 'john@example.com',
    age: 25,
    birthdate: '1998-05-15',
    preferences: { newsletter: true, theme: 'dark' },
    tags: ['developer', 'typescript'],
  });
  console.log('Valid!', validUser);
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log('Validation errors:', error.errors);
    // error.errors: Array of { path: string[], message: string }
  }
}

// 4. Safe Parse (without throwing)
const result = userSchema.safeParse(someData);
if (result.success) {
  const user = result.data; // Type-safe
} else {
  const errors = result.error.errors;
}

// 5. Partial Validation
const partialUserSchema = userSchema.partial();
// All fields optional

const userUpdateSchema = userSchema.pick({ name: true, email: true });
// Only name and email

const requiredUserSchema = userSchema.required();
// Even optional fields become required

// 6. Custom Validation
const passwordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .refine(
    (password) => /[A-Z]/.test(password),
    'Password must contain at least one uppercase letter'
  )
  .refine(
    (password) => /[0-9]/.test(password),
    'Password must contain at least one number'
  );

const confirmPasswordSchema = z.object({
  password: passwordSchema,
  confirmPassword: z.string(),
}).refine(
  (data) => data.password === data.confirmPassword,
  {
    message: "Passwords don't match",
    path: ["confirmPassword"], // Where to show error
  }
);

// 7. Complex Schemas with Transform
const trimmedStringSchema = z.string().transform((val) => val.trim());

const userWithFullNameSchema = userSchema.extend({
  fullName: z.string().transform((val) => 
    val.split(' ').map(word => 
      word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
    ).join(' ')
  ),
});

// 8. Form Integration Pattern
function useZodForm<T extends z.ZodType<any, any>>(schema: T) {
  type FormData = z.infer<T>;
  
  const [data, setData] = useState<FormData>(
    schema.parse({}) // Start with empty validated object
  );
  const [errors, setErrors] = useState<z.ZodError | null>(null);
  
  const validate = useCallback(() => {
    const result = schema.safeParse(data);
    setErrors(result.success ? null : result.error);
    return result.success;
  }, [data, schema]);
  
  const handleSubmit = useCallback((onSubmit: (data: FormData) => void) => {
    if (validate()) {
      onSubmit(data);
    }
  }, [data, validate]);
  
  return { data, setData, errors, validate, handleSubmit };
}

// 9. Integration with Form Libraries
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';

function UserForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      age: 18,
      preferences: { newsletter: false, theme: 'auto' },
      tags: [],
    },
  });
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
      {/* ... */}
    </form>
  );
}
```

---

## **3️⃣ React Hook Form - The Performant Form Library**

### **Concept (70%)**
React Hook Form is a **performant, flexible form library** with **minimal re-renders**. It uses uncontrolled components and refs for better performance.

**React Hook Form Philosophy:**
- ⚡ **Performance first** - Minimal re-renders
- 🎯 **Uncontrolled components** - Use refs instead of state
- 📦 **Tiny bundle** (~12KB)
- 🔧 **Flexible** - Works with any UI library
- 🎭 **Composable** - Build complex forms easily

**Key Features:**
1. **Register API** - Connect inputs to form state
2. **Form state** - `errors`, `touched`, `isDirty`, `isValid`
3. **Validation** - Built-in + schema validation (Zod, Yup)
4. **Form context** - Access form state anywhere
5. **Dev tools** - Visualize form state

### **Code (30%)**
```tsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

// 1. Basic Usage
function BasicForm() {
  const { 
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isValid, isDirty },
    reset,
    watch,
    setValue,
    getValues,
  } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      age: 18,
      subscribe: false,
    },
  });
  
  // Watch specific field (re-renders when field changes)
  const firstName = watch('firstName');
  
  const onSubmit = async (data) => {
    console.log(data);
    await submitToAPI(data);
    reset(); // Reset form after submission
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>First Name</label>
        <input 
          {...register('firstName', {
            required: 'First name is required',
            minLength: { value: 2, message: 'Min 2 characters' },
          })} 
        />
        {errors.firstName && (
          <span className="error">{errors.firstName.message}</span>
        )}
      </div>
      
      <div>
        <label>Last Name</label>
        <input 
          {...register('lastName', {
            required: 'Last name is required',
          })} 
        />
        {errors.lastName && (
          <span className="error">{errors.lastName.message}</span>
        )}
      </div>
      
      <div>
        <label>Email</label>
        <input 
          type="email"
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
              message: 'Invalid email address',
            },
          })} 
        />
        {errors.email && (
          <span className="error">{errors.email.message}</span>
        )}
      </div>
      
      <button 
        type="submit" 
        disabled={isSubmitting || !isValid || !isDirty}
      >
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
      
      <button type="button" onClick={() => reset()}>
        Reset
      </button>
    </form>
  );
}

// 2. With Zod Schema Validation
const schema = z.object({
  username: z.string().min(3),
  email: z.string().email(),
  age: z.number().min(18),
});

function ZodForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      username: '',
      email: '',
      age: 18,
    },
  });
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input {...register('username')} />
      {errors.username && <span>{errors.username.message}</span>}
      {/* ... */}
    </form>
  );
}

// 3. Controller Component (for controlled components)
function ControlledForm() {
  const { control, handleSubmit } = useForm();
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <Controller
        name="firstName"
        control={control}
        rules={{ required: true }}
        render={({ field, fieldState }) => (
          <div>
            <input {...field} />
            {fieldState.error && <span>This field is required</span>}
          </div>
        )}
      />
      
      {/* With Material UI */}
      <Controller
        name="select"
        control={control}
        render={({ field }) => (
          <Select
            {...field}
            options={[
              { value: 'chocolate', label: 'Chocolate' },
              { value: 'strawberry', label: 'Strawberry' },
            ]}
          />
        )}
      />
    </form>
  );
}

// 4. Complex Form with Arrays
function ArrayForm() {
  const { register, control, handleSubmit } = useForm({
    defaultValues: {
      users: [{ name: '', email: '' }],
    },
  });
  
  const { fields, append, remove } = useFieldArray({
    control,
    name: "users",
  });
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`users.${index}.name`)} />
          <input {...register(`users.${index}.email`)} />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}
      
      <button 
        type="button" 
        onClick={() => append({ name: '', email: '' })}
      >
        Add User
      </button>
    </form>
  );
}

// 5. Conditional Fields
function ConditionalForm() {
  const { register, watch, handleSubmit } = useForm();
  
  const showAddress = watch('hasAddress'); // Watch checkbox
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <label>
        <input type="checkbox" {...register('hasAddress')} />
        Add shipping address
      </label>
      
      {showAddress && (
        <>
          <input {...register('address.street')} placeholder="Street" />
          <input {...register('address.city')} placeholder="City" />
          <input {...register('address.zip')} placeholder="ZIP" />
        </>
      )}
    </form>
  );
}

// 6. Async Validation & Submission
function AsyncForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm();
  
  const checkEmail = async (email: string) => {
    const response = await fetch(`/api/check-email?email=${email}`);
    const { available } = await response.json();
    return available || 'Email already taken';
  };
  
  const onSubmit = async (data) => {
    try {
      await submitForm(data);
    } catch (error) {
      setError('root', {
        type: 'manual',
        message: error.message,
      });
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', {
          required: 'Email is required',
          validate: {
            checkEmail: async (email) => await checkEmail(email),
          },
        })}
      />
      {errors.email && <span>{errors.email.message}</span>}
      
      {errors.root && <div className="error">{errors.root.message}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}

// 7. File Upload
function FileForm() {
  const { register, handleSubmit } = useForm();
  
  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input
        type="file"
        {...register('files', {
          validate: {
            fileSize: (files) => {
              if (!files[0]) return true;
              return files[0].size < 5000000 || 'File too large (max 5MB)';
            },
            fileType: (files) => {
              if (!files[0]) return true;
              return ['image/jpeg', 'image/png'].includes(files[0].type) 
                || 'Only JPEG/PNG allowed';
            },
          },
        })}
        multiple
      />
    </form>
  );
}
```

---

## **4️⃣ Formik - The Declarative Form Library**

### **Concept (70%)**
Formik is a **declarative form library** that uses **controlled components**. It's more **verbose but explicit** compared to React Hook Form.

**Formik Philosophy:**
- 📝 **Declarative API** - Clear what's happening
- 🔧 **Built-in validation** - Yup integration out of the box
- 🎯 **Form context** - Access form state anywhere
- 📚 **Good documentation** - Lots of examples

**Formik vs React Hook Form:**
- **Performance**: RHF is faster (uncontrolled)
- **Learning curve**: Formik is easier to understand
- **Bundle size**: Similar
- **API style**: Formik is more explicit/verbose

### **Code (30%)**
```tsx
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

// 1. Basic Formik Form
function BasicForm() {
  const initialValues = {
    firstName: '',
    lastName: '',
    email: '',
  };
  
  const validationSchema = Yup.object({
    firstName: Yup.string()
      .min(2, 'Too Short!')
      .max(50, 'Too Long!')
      .required('Required'),
    lastName: Yup.string()
      .min(2, 'Too Short!')
      .max(50, 'Too Long!')
      .required('Required'),
    email: Yup.string()
      .email('Invalid email')
      .required('Required'),
  });
  
  const handleSubmit = (values, { setSubmitting, resetForm }) => {
    setTimeout(() => {
      console.log(values);
      setSubmitting(false);
      resetForm();
    }, 400);
  };
  
  return (
    <Formik
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ isSubmitting, isValid, dirty }) => (
        <Form>
          <div>
            <label htmlFor="firstName">First Name</label>
            <Field 
              name="firstName" 
              type="text"
              placeholder="Jane"
            />
            <ErrorMessage name="firstName" component="div" />
          </div>
          
          <div>
            <label htmlFor="lastName">Last Name</label>
            <Field 
              name="lastName" 
              type="text"
              placeholder="Doe"
            />
            <ErrorMessage name="lastName" component="div" />
          </div>
          
          <div>
            <label htmlFor="email">Email</label>
            <Field 
              name="email" 
              type="email"
              placeholder="jane@acme.com"
            />
            <ErrorMessage name="email" component="div" />
          </div>
          
          <button 
            type="submit" 
            disabled={isSubmitting || !isValid || !dirty}
          >
            Submit
          </button>
        </Form>
      )}
    </Formik>
  );
}

// 2. Custom Field Components
function CustomField({ label, ...props }) {
  return (
    <div>
      <label htmlFor={props.name}>{label}</label>
      <Field {...props} />
      <ErrorMessage name={props.name}>
        {msg => <div className="error">{msg}</div>}
      </ErrorMessage>
    </div>
  );
}

// 3. Formik with useFormik Hook
function HookForm() {
  const formik = useFormik({
    initialValues: {
      email: '',
      password: '',
    },
    validationSchema: Yup.object({
      email: Yup.string().email('Invalid email').required('Required'),
      password: Yup.string().min(8, 'Too short').required('Required'),
    }),
    onSubmit: values => {
      console.log(values);
    },
  });
  
  return (
    <form onSubmit={formik.handleSubmit}>
      <input
        id="email"
        name="email"
        type="email"
        onChange={formik.handleChange}
        onBlur={formik.handleBlur}
        value={formik.values.email}
      />
      {formik.touched.email && formik.errors.email ? (
        <div>{formik.errors.email}</div>
      ) : null}
      
      <button type="submit">Submit</button>
    </form>
  );
}

// 4. Complex Form with Formik
function ComplexForm() {
  return (
    <Formik
      initialValues={{
        social: {
          facebook: '',
          twitter: '',
        },
        phoneNumbers: [''],
      }}
      onSubmit={console.log}
    >
      {({ values }) => (
        <Form>
          {/* Nested objects */}
          <Field name="social.facebook" placeholder="Facebook" />
          <Field name="social.twitter" placeholder="Twitter" />
          
          {/* Arrays */}
          <FieldArray name="phoneNumbers">
            {({ push, remove }) => (
              <div>
                {values.phoneNumbers.map((_, index) => (
                  <div key={index}>
                    <Field name={`phoneNumbers.${index}`} />
                    <button type="button" onClick={() => remove(index)}>
                      Remove
                    </button>
                  </div>
                ))}
                <button type="button" onClick={() => push('')}>
                  Add Phone Number
                </button>
              </div>
            )}
          </FieldArray>
        </Form>
      )}
    </Formik>
  );
}
```

---

## **📊 Form Library Comparison**

| **Feature** | **React Hook Form** | **Formik** | **Vanilla React** |
|------------|---------------------|------------|------------------|
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| **Bundle Size** | ~12KB | ~13KB | 0KB |
| **Learning Curve** | Medium | Easy | Hard |
| **TypeScript** | Excellent | Good | Manual |
| **Validation** | Flexible (Zod/Yup) | Built-in (Yup) | Manual |
| **Re-renders** | Minimal | Many | Many |
| **Best For** | Complex forms, performance | Simple forms, quick start | Simple forms only |

---

## **🎯 Choosing the Right Solution**

### **Decision Tree:**
```
What are your needs?
├── Performance critical? → Yes → React Hook Form
│
├── Simple form only? → Yes → Formik or vanilla
│
├── Need TypeScript? → Yes → React Hook Form + Zod
│
└── Complex validation? → Yes → React Hook Form + Zod
```

### **Modern Stack Recommendations:**

**1. TypeScript + Validation:**
```typescript
// React Hook Form + Zod
// Why: Type-safe, runtime validation, great performance
```

**2. Quick Prototype:**
```typescript
// Formik + Yup
// Why: Easy to start, good docs, declarative
```

**3. Simple Form Only:**
```typescript
// Controlled components + Zod
// Why: Minimal dependencies, full control
```

**4. Enterprise Application:**
```typescript
// React Hook Form + Zod + TypeScript
// Why: Type safety, performance, scalability
```

---

## **🚀 Advanced Patterns**

### **1. Multi-step Form Wizard**
```tsx
import { useForm, FormProvider } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const step1Schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

const step2Schema = z.object({
  address: z.string().min(5),
  city: z.string().min(2),
});

const fullSchema = step1Schema.merge(step2Schema);

function MultiStepForm() {
  const [step, setStep] = useState(1);
  
  const methods = useForm({
    resolver: zodResolver(fullSchema),
    mode: 'onTouched',
  });
  
  const { handleSubmit, trigger } = methods;
  
  const nextStep = async () => {
    let isValid = false;
    
    if (step === 1) {
      isValid = await trigger(['name', 'email']);
    } else if (step === 2) {
      isValid = await trigger(['address', 'city']);
    }
    
    if (isValid) {
      setStep(s => s + 1);
    }
  };
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={handleSubmit(console.log)}>
        {step === 1 && (
          <Step1 onNext={nextStep} />
        )}
        {step === 2 && (
          <Step2 onNext={nextStep} />
        )}
        {step === 3 && (
          <ReviewStep onSubmit={handleSubmit(console.log)} />
        )}
      </form>
    </FormProvider>
  );
}

function Step1({ onNext }) {
  const { register, formState: { errors } } = useFormContext();
  
  return (
    <div>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
      
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <button type="button" onClick={onNext}>Next</button>
    </div>
  );
}
```

### **2. Dynamic Schema Based on User Input**
```tsx
function DynamicForm() {
  const [userType, setUserType] = useState<'individual' | 'business'>('individual');
  
  const schema = useMemo(() => {
    const baseSchema = z.object({
      name: z.string().min(2),
      userType: z.enum(['individual', 'business']),
    });
    
    if (userType === 'business') {
      return baseSchema.extend({
        companyName: z.string().min(2),
        taxId: z.string().regex(/^\d{9}$/, 'Invalid tax ID'),
      });
    }
    
    return baseSchema.extend({
      ssn: z.string().regex(/^\d{3}-\d{2}-\d{4}$/, 'Invalid SSN'),
    });
  }, [userType]);
  
  const methods = useForm({
    resolver: zodResolver(schema),
  });
  
  return (
    <FormProvider {...methods}>
      <form>
        <select {...methods.register('userType')}>
          <option value="individual">Individual</option>
          <option value="business">Business</option>
        </select>
        
        {userType === 'business' ? (
          <>
            <input {...methods.register('companyName')} />
            <input {...methods.register('taxId')} />
          </>
        ) : (
          <input {...methods.register('ssn')} />
        )}
      </form>
    </FormProvider>
  );
}
```

### **3. Real-time API Validation**
```tsx
function RealTimeValidationForm() {
  const debouncedCheckEmail = useDebouncedCallback(
    async (email: string) => {
      const response = await fetch(`/api/validate-email?email=${email}`);
      return response.ok;
    },
    500
  );
  
  const methods = useForm({
    defaultValues: { email: '' },
    mode: 'onChange',
  });
  
  const { register, formState: { errors } } = methods;
  
  return (
    <form>
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email',
          },
          validate: {
            isAvailable: async (email) => {
              if (!email) return true;
              const isAvailable = await debouncedCheckEmail(email);
              return isAvailable || 'Email already taken';
            },
          },
        })}
      />
      {errors.email && <span>{errors.email.message}</span>}
    </form>
  );
}
```

---

## **📋 Best Practices**

### **TypeScript:**
1. ✅ **Define interfaces** for all form data
2. ✅ **Use generics** for reusable form components
3. ✅ **Create utility types** for errors, touched states
4. ✅ **Implement type guards** for runtime validation
5. ❌ **Don't use `any`** - Be specific with types

### **Zod:**
1. ✅ **Define schema once** - Get both types and validation
2. ✅ **Use safeParse** - Don't throw errors unnecessarily
3. ✅ **Compose schemas** - Reuse validation logic
4. ✅ **Add descriptive error messages**
5. ❌ **Don't duplicate validation logic** elsewhere

### **React Hook Form:**
1. ✅ **Use uncontrolled components** when possible
2. ✅ **Enable mode: 'onTouched'** for better UX
3. ✅ **Use `Controller`** for controlled components
4. ✅ **Lazy validate** with `trigger`
5. ✅ **Reset form** after successful submission
6. ❌ **Don't overuse `watch`** - it causes re-renders

### **Formik:**
1. ✅ **Use Yup schemas** for validation
2. ✅ **Wrap in `Formik`** context provider
3. ✅ **Use `Field` components** for consistency
4. ✅ **Handle dirty/pristine states**
5. ❌ **Don't put everything in one component**

---

## **🎓 Key Takeaways:**

1. **TypeScript**: For type safety and better DX
2. **Zod**: For single source of truth (types + validation)
3. **React Hook Form**: For performance and flexibility
4. **Formik**: For declarative, easy-to-understand forms

**Modern Form Stack 2024:**
```typescript
// Recommended stack for most projects:
// React Hook Form + Zod + TypeScript

// Why?
// ⚡ Performance - Minimal re-renders
// 🛡️ Type Safety - Full TypeScript support  
// 🔄 Validation - Runtime + compile time
// 📦 Bundle Size - Reasonable footprint

// Alternative for simple forms:
// Formik + Yup (easier to start)
```

**Remember**: Good forms are **fast, accessible, and provide clear feedback**. Always validate on the server too - client validation is for UX, not security! 🚀